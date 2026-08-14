
# Plan: Write-up of the inactivity → release pipeline (current state, fixes made, and open gaps)

## Context

Over this session we did a deep multi-round review (CodeRabbit findings → multi-agent code review → manual follow-up) of Anchora's inactivity → release pipeline (`anchor-backend/src/modules/{inactivity,release,executor}/`), fixed a long list of correctness bugs, and — in the last few exchanges — talked through how thoroughly the app currently follows up with trusted contacts after a release fires, and found several gaps and pieces of dead/unwired code. The user wants this captured as a lasting reference document: how the pipeline works today, what was fixed this session, and what's still open — both things we explicitly flagged and things worth flagging that we haven't discussed yet.

No further codebase exploration is needed for this — every fact below was established via direct `Read`/`Bash` inspection earlier in this same session (file:line citations throughout the conversation). This plan is a writing/synthesis task, not a code-implementation task.

## Deliverable

A single markdown file: **`anchor-backend/docs/inactivity-release-pipeline.md`**

Rationale: matches this repo's existing docs convention (`anchor-backend/docs/AWS_KMS_SETUP.md`), lives with the code it describes, easy to keep in sync in future sessions, and is git-versioned so its accuracy over time is checkable via blame/diff.

## Document outline

**1. Overview** One paragraph: Anchora is a "dead man's switch" vault app; this doc covers the inactivity → release subsystem specifically (not auth, billing, vault encryption, etc.).

**2. The state machine** `InactivityStatus`: `ACTIVE → NOTIFIED → RELEASING`. Table of transitions with triggers/waits (mirrors the table already in `anchor-backend/CLAUDE.md`, but with full explanation and file:line pointers into `inactivity.service.ts`'s `processInactivityForUser`). Explicitly cover the now-added `RELEASING` branch and why it's a conditional retry, not an unconditional one.

**3. Executor (trusted contact) model** Multi-contact, plan-gated limits (FREE=1/PRO=3), `removedAt` soft-delete semantics, case-insensitive email uniqueness (raw migration index), revive-on-re-add behavior in `createExecutor`. Reference `executor.service.ts`, `plan-limits.ts`.

**4. `initiateRelease` walkthrough** Step-by-step of the current (post-fix) implementation: existing-release resume/retry logic, first-contact-notify loop (with the self-accept skip), the transaction + its try/catch revert-to-NOTIFIED, report generation try/catch (resume-on-failure), post-completion notify loop (log-only on failure). Reference exact file:line in `release.service.ts`.

**5. Report generation & access gating** `report.service.ts` (generation, no identity gating) vs `report-access.service.ts` (download gating: must be an accepted contact, verified govId, within the 30-day access window from `triggeredAt`). Note the hard cutoff with no warning.

**6. Notification/reminder inventory** Full table of every executor/release-related notification method, when it fires, and — critically — when it _stops_ firing. This directly captures the gap analysis from the last two exchanges: pre-release nudges stop the instant `RELEASING` begins; exactly one notification at release completion; nothing afterward until the 30-day window silently closes.

**7. What was fixed this session (changelog)** Bullet list, each with a one-line "why": soft-delete conversion (`removeExecutor`/`exitEstate`), `removedAt` filtering swept across executor/admin reads, case-insensitive email uniqueness + revive-on-create, `notifyExecutor`'s persist-after-send-not-before fix, the release-creation transaction's try/catch, the `RELEASING` retry branch, shared helpers extracted (`findOwnedExecutor`, `assertNoActiveRelease`, `assertTrustedContactLimit`/`assertDocumentLimit`, `findAccountEmailSet`).

**8. Known gaps — already discussed**

- No advisory lock around `initiateRelease`'s existing-release check-then-create — same class of race as the one fixed in `createExecutor`, not yet applied here. Concretely reachable if the admin dev-tools trigger and the nightly cron ever overlap for the same user.
- No reminder system after release completes, and no warning before the 30-day access window expires.
- Dead code suggesting an unfinished feature: `ReportService.checkAndGeneratePendingReports` + `notifyReportNowAvailableExecutor`, and four `notifyVerification*Executor` methods — all fully written, never called. The admin gov-ID-verification-approval endpoint they depend on doesn't exist yet.

**9. Additional considerations — not yet discussed** New points worth surfacing (to be written out fully in the doc, not just this outline):

- Report generation's dependency chain (Puppeteer → S3/R2 → KMS decrypt) is a real multi-point failure surface with no alerting/observability when a release gets stuck in `PENDING` — an admin would have to know to look.
- No test coverage exists yet for the concurrent-double-`initiateRelease` race (item 8's first bullet) — only reasoned about, not verified.
- Unclear what happens to report files in S3/R2 after the 30-day access window closes — worth confirming whether they're deleted or linger indefinitely (retention/compliance question).
- Once `RELEASING` begins, does the _owner_ (if not actually gone — false-positive case) get any visibility that a release is in progress, beyond the ability to check in and cancel it? Worth checking whether there's an in-app indicator.
- The 02:00 UTC nightly cron means all timing (grace periods, nudge cadence, retry cadence) is bucketed to once-per-day resolution — worth noting as an inherent latency floor, not a bug.
- No download audit trail called out for report access (compliance-relevant for an inheritance/estate product) — worth checking if `getReport()` logs access anywhere.

**10. Sourcing note** State clearly at the top of the doc that it reflects the codebase as of this session's fixes (reference the commit these fixes will land in) and should be spot-checked against the code before being relied on in a future session, consistent with how living documentation ages.

## Verification

Documentation-only change — verification is a read-through for accuracy against the current code, not a test run:

- Cross-check every file:line reference in the doc against the actual current `anchor-backend/src/modules/{inactivity,release,executor}/*.ts` and `report-access.service.ts` after writing.
- Confirm the notification-method inventory (section 6) matches `grep -n "async notify" src/modules/notification/notification.service.ts` exactly, including which are and aren't called anywhere (already established this session via grep, but re-verify at write time in case anything changed).

Add Comment