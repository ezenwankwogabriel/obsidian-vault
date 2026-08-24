# Anchora — Error Logging & Handling Spec

## Current State

Anchora backend is **NestJS** (not Next.js API routes). Logging currently uses NestJS's built-in `Logger` module + DB writes. The gaps:

- **No real-time alerting** — errors sit in the DB until someone queries
- **DB is a SPOF for observability** — if Postgres is down, logs are lost
- **No structured severity routing** — everything is flat, no way to separate "investigate now" from "fine, expected"
- **No dedup/grouping** — a flood fills the DB and burns connections
- **No trace context** — can't follow a request through the stack

## Target Architecture

```
NestJS Application
    │
    ├── NestJS Logger → structured JSON → stdout (Railway captures)
    │       │
    │       └── Railway Logs (free, 7-day retention)
    │
    ├── ExceptionFilter (global) → Sentry
    │       │
    │       └── Real-time alerts → Telegram
    │
    └── DB audit_events (business events only, not system errors)
```

### Why This Stack

| Component | Why | Cost |
|-----------|-----|------|
| **Railway Logs** | Already there. stdout is auto-captured. 0 setup. | Free |
| **Sentry** | Error dedup, grouping, release tracking, breadcrumbs. `@sentry/nestjs` exists. | $0–$26/mo |
| **NestJS Logger** (kept) | Already in use. Just needs config + a custom transport to ship to Sentry. No new dep. | Free |
| **DB audit_events** | Keep for business-audit: release triggers, contact acceptance, billing. Not errors. | Already exists |

## Implementation Plan

### Phase 1 — Structured Logging with NestJS Logger (1–2 hours)

NestJS's `Logger` is fine — no need to replace it with pino/Winston. What matters is making it output structured JSON in production.

**Step 1: Custom logger service that wraps NestJS Logger**

```typescript
// src/logging/structured-logger.service.ts
import { Injectable, LoggerService, LogLevel } from '@nestjs/common';
import * as Sentry from '@sentry/nestjs';

@Injectable()
export class StructuredLoggerService implements LoggerService {
  private context?: string;

  setContext(context: string) {
    this.context = context;
  }

  private formatMessage(level: string, message: any, optionalParams: any[]) {
    const entry: Record<string, unknown> = {
      timestamp: new Date().toISOString(),
      level,
      context: this.context,
      message: typeof message === 'string' ? message : JSON.stringify(message),
    };

    // Flatten optionalParams into the entry (they're context objects in practice)
    if (optionalParams.length > 0) {
      const lastParam = optionalParams[optionalParams.length - 1];
      if (typeof lastParam === 'object' && lastParam !== null && !(lastParam instanceof Error)) {
        Object.assign(entry, lastParam);
      }
    }

    return entry;
  }

  log(message: any, ...optionalParams: any[]) {
    const entry = this.formatMessage('log', message, optionalParams);
    console.log(JSON.stringify(entry));
  }

  error(message: any, ...optionalParams: any[]) {
    const entry = this.formatMessage('error', message, optionalParams);
    console.error(JSON.stringify(entry));

    // Ship to Sentry if it looks like an actual error
    const errorObj = optionalParams.find(p => p instanceof Error) as Error | undefined;
    if (errorObj || typeof message === 'string') {
      Sentry.withScope(scope => {
        scope.setContext('log', entry);
        Sentry.captureException(errorObj || new Error(message));
      });
    }
  }

  warn(message: any, ...optionalParams: any[]) {
    const entry = this.formatMessage('warn', message, optionalParams);
    console.warn(JSON.stringify(entry));
  }

  debug(message: any, ...optionalParams: any[]) {
    const entry = this.formatMessage('debug', message, optionalParams);
    console.debug(JSON.stringify(entry));
  }

  verbose(message: any, ...optionalParams: any[]) {
    const entry = this.formatMessage('verbose', message, optionalParams);
    console.debug(JSON.stringify(entry));
  }
}
```

**Step 2: Register it in `main.ts`**

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { StructuredLoggerService } from './logging/structured-logger.service';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new StructuredLoggerService(),
  });
  // ...
}
bootstrap();
```

**Step 3: Usage in services — no code changes needed**

NestJS Logger already works via injection. The existing pattern stays:

```typescript
// Existing pattern — keeps working, now outputs structured JSON
constructor(private readonly logger: Logger) {}

someMethod() {
  this.logger.log(`Processing release for user ${userId}`);
  this.logger.error('Payment webhook failed', error.stack, { userId, reference });
}
```

The only change is `main.ts` registering the custom logger. All existing `this.logger.log()` calls automatically become structured.

**Redact sensitive fields:**

```typescript
// src/logging/redact.ts
const SENSITIVE_KEYS = [
  'password', 'token', 'secret', 'authorization',
  'nin', 'bvn', 'selfie', 'accessToken', 'refreshToken',
];

export function redact(obj: Record<string, unknown>): Record<string, unknown> {
  const result: Record<string, unknown> = {};
  for (const [key, value] of Object.entries(obj)) {
    if (SENSITIVE_KEYS.some(k => key.toLowerCase().includes(k.toLowerCase()))) {
      result[key] = '[REDACTED]';
    } else if (typeof value === 'object' && value !== null && !(value instanceof Date)) {
      result[key] = redact(value as Record<string, unknown>);
    } else {
      result[key] = value;
    }
  }
  return result;
}
```

### Phase 2 — Global Exception Filter + Sentry (1–2 hours)

```bash
npm install @sentry/nestjs
```

**Step 1: Initialize Sentry**

```typescript
// src/instrument.ts (auto-generated by `npx @sentry/wizard -i nestjs`)
import * as Sentry from '@sentry/nestjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  // Ignore these — they're client errors, not bugs
  ignoreErrors: [
    'UnauthorizedException',
    'NotFoundException',
    'BadRequestException',
    'ForbiddenException',
  ],
});
```

**Step 2: Global exception filter (catches everything Sentry doesn't auto-catch)**

```typescript
// src/logging/sentry-exception.filter.ts
import {
  ExceptionFilter,
  Catch,
  ArgumentsHost,
  HttpException,
  HttpStatus,
} from '@nestjs/common';
import * as Sentry from '@sentry/nestjs';

@Catch()
export class SentryExceptionFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const request = ctx.getRequest();

    // Don't ship 4xx to Sentry — they're client errors
    const status =
      exception instanceof HttpException
        ? exception.getStatus()
        : HttpStatus.INTERNAL_SERVER_ERROR;

    if (status >= 500) {
      Sentry.withScope(scope => {
        scope.setExtra('url', request.url);
        scope.setExtra('method', request.method);
        scope.setExtra('userId', request.user?.id);
        Sentry.captureException(exception);
      });
    }

    response.status(status).json({
      statusCode: status,
      message:
        exception instanceof HttpException
          ? exception.message
          : 'Internal server error',
    });
  }
}
```

**Step 3: Register filter globally**

```typescript
// src/main.ts
import { SentryExceptionFilter } from './logging/sentry-exception.filter';

async function bootstrap() {
  const app = await NestFactory.create(AppModule, {
    logger: new StructuredLoggerService(),
  });

  app.useGlobalFilters(new SentryExceptionFilter());
  // ...
}
```

**What goes to Sentry vs. what doesn't:**

| Event | Sentry? | DB? | Railway Logs? |
|-------|---------|-----|---------------|
| Uncaught exception | ✅ Auto | ❌ | ✅ |
| Route 500 | ✅ Filter | ❌ | ✅ |
| Payment failure (Paystack) | ✅ Manual | ✅ Audit | ✅ |
| QoreID verification failure | ✅ Manual | ❌ | ✅ |
| Cron job crash | ✅ Manual | ❌ | ✅ |
| Rate limit hit | ❌ | ❌ | ✅ (warn) |
| User not found (404) | ❌ | ❌ | ✅ (debug) |
| Validation error (400) | ❌ | ❌ | ✅ (debug) |
| Release triggered | ❌ | ✅ Audit | ✅ (info) |
| Contact accepted | ❌ | ✅ Audit | ✅ (info) |
| Unauthorized (401) | ❌ | ❌ | ❌ |

### Phase 3 — Real-time Alerts (30 min)

Sentry → Telegram.

1. Create a Telegram channel/group for alerts
2. Sentry: Settings → Integrations → Telegram → Connect
3. Configure alert rule:

```
Alert: "Anchora Production Errors"
Condition: New issue (first occurrence) OR issue regressed
Priority: Immediate
Route to: Telegram
```

### Phase 4 — Request Tracing (future, post-MVP)

NestJS interceptor instead of Next.js middleware:

```typescript
// src/logging/request-id.interceptor.ts
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common';
import { Observable } from 'rxjs';
import { v4 as uuid } from 'uuid';
import { Logger } from '@nestjs/common';

@Injectable()
export class RequestIdInterceptor implements NestInterceptor {
  private readonly logger = new Logger('RequestId');

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const requestId = request.headers['x-request-id'] || uuid();
    request.requestId = requestId;

    this.logger.log(`→ ${request.method} ${request.url}`, { requestId });
    return next.handle();
  }
}
```

Apply globally in `main.ts`:

```typescript
app.useGlobalInterceptors(new RequestIdInterceptor());
```

## What to Keep / What to Change

### Keep — DB Audit Table

Rename logs → `audit_events`, narrow scope to business events only:

```sql
CREATE TABLE audit_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type TEXT NOT NULL,        -- 'release.triggered', 'contact.accepted', 'billing.subscription'
  actor_id UUID,
  target_id UUID,
  details JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

```typescript
// Inserted only for business-significant moments, not errors
await this.db.insert(auditEvents).values({
  eventType: 'release.triggered',
  actorId: userId,
  targetId: releaseId,
  details: { contactCount: 3, plan: 'PRO' },
});
```

### Kill — DB Error Logs

Stop writing system errors, stack traces, HTTP 500 details to DB. Those go to Sentry + Railway Logs.

### Cron Job Monitoring

```typescript
// In cron handlers (e.g., inactivity checker)
import * as Sentry from '@sentry/nestjs';

async handleCron() {
  const checkInId = Sentry.captureCheckIn({
    monitorSlug: 'inactivity-check',
    status: 'in_progress',
  });

  try {
    await this.inactivityService.run();
    Sentry.captureCheckIn({ checkInId, monitorSlug: 'inactivity-check', status: 'ok' });
  } catch (err) {
    Sentry.captureCheckIn({ checkInId, monitorSlug: 'inactivity-check', status: 'error' });
    throw err;
  }
}
```

Sentry alerts if a cron: crashes, misses its schedule, or takes too long.

## Error Handling Patterns (NestJS-specific)

### Fail Loudly, Fail Early

```typescript
// Good — 401 is expected (no alert), 500 ships to Sentry
async processPayment(userId: string) {
  const user = await this.usersService.findById(userId);
  if (!user) {
    throw new NotFoundException('User not found'); // 404 → no Sentry
  }

  try {
    const result = await this.paystack.charge(user, amount);
    return result;
  } catch (err) {
    // 500 because Paystack failing is unexpected
    this.logger.error('Paystack charge failed', err.stack, { userId, amount });
    throw new InternalServerErrorException('Payment processing failed');
  }
}
```

### Don't Catch What You Can't Handle

```typescript
// Bad — swallows, nobody knows
try {
  await this.notifyContacts(releaseId);
} catch (err) {
  this.logger.log('notification failed', err); // lost
}

// Good — let it bubble to the exception filter
await this.notifyContacts(releaseId);
```

### Catch at the Boundary

One try/catch per controller method or cron handler. Don't catch-rethrow in every service method.

```typescript
@Post('subscribe')
async subscribe(@Body() dto: SubscribeDto, @Req() req: AuthenticatedRequest) {
  try {
    return await this.billingService.subscribe(req.user.id, dto.plan);
  } catch (err) {
    this.logger.error('subscribe failed', err.stack, {
      userId: req.user.id,
      plan: dto.plan,
    });
    throw new InternalServerErrorException();
  }
}
```

## Implementation Order

1. **Create `StructuredLoggerService`** — wraps NestJS Logger, outputs JSON, ships `error()` to Sentry (1 hr)
2. **Register in `main.ts`** — one-line change, all existing `this.logger.*` calls work (5 min)
3. **Add `@sentry/nestjs` + global exception filter** — catches unhandled exceptions (1 hr)
4. **Wire Telegram alerts** — Sentry integration (30 min)
5. **Narrow DB logs** → `audit_events` (1 hr)
6. **Add cron monitoring** — Sentry check-ins per cron (30 min)
7. **Request IDs** — interceptor (optional, post-MVP)

## Costs

| Service | Free Tier | Paid Tier |
|---------|-----------|-----------|
| Sentry | 5K errors/mo, 1 user, 7-day history | $26/mo (50K errors, unlimited users) |
| Railway Logs | Included | Included |
| Telegram alerts | Free | Free |

## Pitfalls

1. **Don't log PII** — `this.logger.log(user)` will dump the full user object. Pass specific fields only: `this.logger.log('user action', { userId: user.id })`.
2. **Don't alert on 4xx** — 400/401/403/404 are client errors. The exception filter already routes only 5xx to Sentry.
3. **No alert fatigue** — Sentry groups identical errors. One Telegram message per unique issue, not per occurrence.
4. **DB-down resilience** — stdout logging survives DB outages. DB audit events are best-effort only.
5. **Railway log retention is 7 days** — for longer, ship to Sentry (errors) or Logtail/Axiom (all logs) at scale. Not needed at MVP.