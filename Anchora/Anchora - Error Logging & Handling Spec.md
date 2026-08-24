# Anchora — Error Logging & Handling Spec

## Current State

Anchora logs to the database (`logs` table or similar). This works for post-mortem inspection but has critical gaps for production:

- **No real-time alerting** — you only discover errors when you query the DB
- **DB is a SPOF for observability** — if the DB is down, logs are lost and you're blind to why
- **No structured severity** — everything is flat rows, hard to filter by urgency
- **No rate-limiting or dedup** — a flood of identical errors fills the table and burns DB connections
- **No trace context** — can't tie a log line to a specific user request through the stack

## Target Architecture

```
Application (Next.js API routes, cron jobs, background tasks)
    │
    ├── Structured JSON logs → stdout (Railway captures automatically)
    │       │
    │       └── Railway Logs (built-in, free, 7-day retention)
    │
    ├── Error-level events → Sentry (or equivalent)
    │       │
    │       └── Real-time alerts → Telegram / Email / Slack
    │
    └── DB logs (keep for business-audit events only, not system errors)
```

### Why This Stack

| Component | Why | Cost |
|-----------|-----|------|
| **Railway Logs** | Already there. `console.log` → stdout is auto-captured. 0 setup. | Free (included) |
| **Sentry** | Industry standard for error tracking. De-duplication, release tracking, source maps, breadcrumbs. Free tier: 5K errors/month. | $0–$26/mo |
| **pino** (logger) | Fastest Node.js JSON logger. 30% faster than Winston. Built for structured logging. | Free (OSS) |
| **DB logs** | Keep for audit trail only: release triggers, contact acceptance, billing events. Not system errors. | Already exists |

## Implementation Plan

### Phase 1 — Structured Logging (1–2 hours)

Replace ad-hoc `console.log` with `pino` across the codebase.

```typescript
// lib/logger.ts
import pino from 'pino';

const isProduction = process.env.NODE_ENV === 'production';
const isTest = process.env.NODE_ENV === 'test';

export const logger = pino({
  level: isProduction ? 'info' : 'debug',
  // In production: machine-parseable JSON to stdout (Railway captures)
  // In development: human-readable with timestamps
  transport: isProduction || isTest
    ? undefined
    : {
        target: 'pino-pretty',
        options: { colorize: true, translateTime: 'HH:MM:ss' },
      },
  // Redact sensitive fields from logs
  redact: {
    paths: [
      'password', 'token', 'secret', 'authorization',
      'headers.authorization', 'headers.cookie',
      'body.password', 'body.token',
      'nin', 'bvn', 'selfie',  // QoreID PII
    ],
    censor: '[REDACTED]',
  },
  // Every log line gets these
  base: {
    env: process.env.NODE_ENV,
    service: 'anchora-api',
  },
});
```

**Usage pattern:**

```typescript
// Before (current pattern — ad-hoc, mixed levels)
console.log('User logged in', userId);
console.error('Payment failed', error);

// After (structured, searchable)
logger.info({ userId }, 'user authenticated');
logger.error({ err: error, userId, reference }, 'payment webhook failed');
logger.warn({ retries, maxRetries }, 'retrying QoreID session');
```

**Key rules:**
- First argument is always a context object (indexed for search)
- Second argument is always a human-readable message
- Never log: passwords, tokens, NIN, BVN, full card numbers, session cookies
- Child loggers for subsystems:

```typescript
const billingLogger = logger.child({ component: 'billing' });
billingLogger.info({ plan: 'PRO', userId }, 'subscription activated');
```

### Phase 2 — Sentry Error Tracking (1–2 hours)

```bash
cd anchora && npm install @sentry/nextjs
```

```typescript
// sentry.config.ts (or next.config.js wrapping)
// Sentry's Next.js SDK auto-instruments:
// - API route errors
// - getServerSideProps / getStaticProps errors
// - Middleware errors
// - Unhandled promise rejections
// - Uncaught exceptions
```

**What goes to Sentry vs. what doesn't:**

| Event | Sentry? | DB? | Railway Logs? |
|-------|---------|-----|---------------|
| Uncaught exception | ✅ Auto | ❌ | ✅ |
| API route 500 | ✅ Auto | ❌ | ✅ |
| Payment failure (Paystack) | ✅ Manual | ✅ Audit | ✅ |
| QoreID verification failure | ✅ Manual | ❌ | ✅ |
| Cron job crash | ✅ Manual | ❌ | ✅ |
| Rate limit hit | ❌ | ❌ | ✅ (warn) |
| User not found (404) | ❌ | ❌ | ✅ (debug) |
| Validation error (400) | ❌ | ❌ | ✅ (debug) |
| Release triggered | ❌ | ✅ Audit | ✅ (info) |
| Contact accepted invite | ❌ | ✅ Audit | ✅ (info) |

**Rule of thumb:** Sentry gets things that need a human to look at them. Not 4xx validation errors, not expected edge cases. 5xx, crashes, integration failures, data corruption.

**Sentry + pino integration:**

```typescript
// lib/logger.ts (add this)
import * as Sentry from '@sentry/nextjs';

// Custom pino transport or level hook that ships ERROR+FATAL to Sentry
export function captureException(err: Error, context?: Record<string, unknown>) {
  Sentry.withScope((scope) => {
    if (context) {
      scope.setContext('details', context);
    }
    Sentry.captureException(err);
  });
  // Also log locally so Railway has it
  logger.error({ err, ...context }, err.message);
}
```

### Phase 3 — Real-time Alerts (30 min)

Sentry → Telegram integration. Errors appear in a Telegram channel within seconds.

1. Create a Telegram channel or group for alerts
2. In Sentry: Settings → Integrations → Telegram → Connect
3. Configure alert rules:

```
Alert: "Anchora Production Errors"
Condition: New issue (first occurrence) OR issue regressed
Priority: Immediate
Route to: Telegram channel
```

**Additional alert channels (optional):**
- **Urgent**: Sentry issue → Telegram (instant)
- **Digest**: Sentry weekly summary email (Sentry auto-sends this)
- **Health check**: Railway has built-in health check + notification if service is down

### Phase 4 — Request Tracing (future, post-MVP)

Add `X-Request-ID` header on every request so you can trace a single user action across all log lines and Sentry events.

```typescript
// middleware.ts
import { v4 as uuid } from 'uuid';

export function middleware(request: NextRequest) {
  const requestId = request.headers.get('x-request-id') || uuid();
  const response = NextResponse.next();
  response.headers.set('x-request-id', requestId);
  return response;
}
```

```typescript
// In any API route:
const requestId = headers().get('x-request-id');
const reqLogger = logger.child({ requestId, userId: session.user.id });
reqLogger.info('processing vault update');
```

Every log line and Sentry event from the same request shares a `requestId` — grep Railway logs for one ID to see the full journey.

## What to Keep / What to Change

### Keep — DB Audit Table

Rename and narrow scope. The current `logs` table becomes `audit_events`:

```sql
CREATE TABLE audit_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type TEXT NOT NULL,        -- 'release.triggered', 'contact.accepted', 'billing.subscription'
  actor_id UUID,                   -- which user caused it
  target_id UUID,                  -- which entity was affected
  details JSONB,                   -- machine-readable context
  created_at TIMESTAMPTZ DEFAULT now()
);
```

Inserted in application code for **business-significant events only**, not system errors:

```typescript
await db.insert(auditEvents).values({
  eventType: 'release.triggered',
  actorId: userId,
  targetId: releaseId,
  details: { contactCount: 3, plan: 'PRO' },
});
```

### Kill — DB Error Logs

Stop writing system errors, stack traces, and HTTP 500 details to the DB. Those go to Sentry + Railway Logs.

### Add — Cron Job Monitoring

Sentry has a "Cron Monitoring" feature. Register each cron as a monitored job:

```typescript
// In your cron handler (e.g., inactivity checker)
const checkInId = Sentry.captureCheckIn({
  monitorSlug: 'inactivity-check',
  status: 'in_progress',
});

try {
  await runInactivityCheck();
  Sentry.captureCheckIn({
    checkInId,
    monitorSlug: 'inactivity-check',
    status: 'ok',
  });
} catch (err) {
  Sentry.captureCheckIn({
    checkInId,
    monitorSlug: 'inactivity-check',
    status: 'error',
  });
  throw err;
}
```

Sentry alerts you if a cron job:
- Crashes (error status)
- Doesn't check in on schedule (missed heartbeat)
- Takes longer than expected (duration threshold)

## Error Handling Patterns

### API Routes: Fail Loudly, Fail Early

```typescript
// Good — specific error, logged once, Sentry gets context
export async function POST(req: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session) {
      // 401 is expected, don't alert
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    const result = await processPayment(session.user.id);
    return NextResponse.json(result);
  } catch (err) {
    // 500 is unexpected, alert
    captureException(err as Error, {
      userId: session?.user?.id,
      endpoint: 'POST /api/billing/subscribe',
    });
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Don't Catch What You Can't Handle

```typescript
// Bad — swallows the error, no one knows it failed
try {
  await notifyContacts(releaseId);
} catch (err) {
  console.log('notification failed', err); // lost in stdout noise
}

// Good — let it bubble to the error boundary, Sentry catches it
await notifyContacts(releaseId);
```

### Handle at the Boundary

Catch only at the edge: API route handler, cron job entry point, queue worker. Don't catch-rethrow in every function.

```typescript
// One try/catch per route, not per function call
export async function POST(req: NextRequest) {
  try {
    // 20 lines of business logic — no try/catch inside
    return NextResponse.json(await doTheThing(req));
  } catch (err) {
    captureException(err as Error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

## Implementation Order

1. **Add `pino` + `lib/logger.ts`** — replace all `console.log`/`console.error` with structured logging (1–2 hrs)
2. **Add Sentry** — `@sentry/nextjs`, auto-instruments everything (1 hr)
3. **Wire Telegram alerts** — Sentry integration (30 min)
4. **Narrow DB logs** — rename to `audit_events`, stop writing system errors, keep business events (1 hr)
5. **Add cron monitoring** — register each cron with Sentry check-ins (30 min)
6. **Add request IDs** — middleware + child loggers (optional, post-MVP)

## Costs

| Service | Free Tier | Paid Tier |
|---------|-----------|-----------|
| Sentry | 5K errors/mo, 1 user, 7-day history | $26/mo (50K errors, unlimited users, 90-day history) |
| Railway Logs | Included in plan | Included |
| pino | Free (OSS) | - |
| Telegram alerts | Free | - |

Start free, upgrade Sentry when you hit 5K errors/month (that's ~160/day — you'll know when you need it).

## Pitfalls

1. **Don't log PII** — pino redaction covers known fields, but `logger.info(userObject)` will leak. Always pass specific fields, never whole objects that contain user data.
2. **Don't alert on 4xx** — 400, 401, 403, 404 are client errors, not system errors. Sentry should ignore these.
3. **Don't create alert fatigue** — one Telegram message per unique error, not per occurrence. Sentry groups identical errors automatically.
4. **Don't lose logs when DB is down** — the new setup logs to stdout (Railway captures even before the app fully boots). DB audit events are best-effort only.
5. **Railway log retention is 7 days** — for longer retention, ship to an external service (Sentry keeps errors; for all logs, consider Logtail/Axiom at scale). Not needed at MVP stage.