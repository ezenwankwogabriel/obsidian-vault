
## Staging test scenarios — inactivity → release pipeline

**Setup**: confirm staging has `DEV_TOOLS_ENABLED=true` (or non-prod `NODE_ENV`) so `/admin/dev/*` and `POST /billing/dev/activate` work, and that outbound email is deliverable/inspectable on staging.

**Key dev tools**:

- `GET /admin/dev/users/state?q=<id|email>` — inspect status/timestamps/active release
- `POST /admin/dev/users/:userId/set-status {status, backdateDays?}` — force ACTIVE/NOTIFIED/RELEASING (RELEASING immediately calls `initiateRelease()`)
- `POST /admin/dev/users/:userId/run-inactivity-check` — real per-user transition logic on demand
- `POST /admin/dev/users/:userId/reset` — cancel + reset to ACTIVE
- `POST /admin/dev/users/:userId/set-plan {plan}` — force FREE/PRO

### A. Core happy path

1. Single accepted contact → NOTIFIED (backdated 8d) → run-check → RELEASING → report generated → contact downloads.
2. Never-notified contact still gets included when release triggers.
3. Contact who self-accepted before ever being notified → first-contact invite skipped, still gets an `ExecutorRelease` row.

### B. Grace period & nudges

4. Within 7-day grace → no nudge.
5. Grace expired → nudges fire every 3 days, not sooner.
6. All contacts decline → resets to ACTIVE.
7. One declines, one pending → status holds (doesn't reset, doesn't advance).
8. Pending contact accepts → advances to RELEASING immediately (equal access).
9. Zero contacts → resets to ACTIVE.

### C. Check-in / cancellation

10. Owner activity during RELEASING cancels the release, resets to ACTIVE, no cooling-off, contacts get cancellation email.
11. Cancelled release's `ExecutorRelease` rows persist (historical record, not deleted).
12. Re-triggering after cancellation creates a fresh `Release` row.
13. Two requests within 60s only trigger one cancel cycle (throttle).

### D. Multi-contact equal access

14. 2nd contact accepted, 1st untouched → 2nd can still access report.
15. 1st (non-accepted) contact denied even though 2nd has access.
16. 3 PRO contacts, mixed decline/accept/no-response → only accepted + never-notified get access.

### E. Report access / identity verification

17. `UNVERIFIED` executor → `403 IDENTITY_VERIFICATION_REQUIRED`.
18. `FAILED` verification → same block, retryable (only `VERIFIED` blocks retry).
19. Verification check takes priority over window-expired/report-not-ready errors.
20. Release >30 days old → `403` access-window-expired.
21. Report not yet generated → `404 REPORT_NOT_READY`.
22. Signed URL >24h old → regenerates transparently.

### F. Plan limits & downgrade (suspected gaps — worth confirming)

23. FREE user blocked adding a 2nd contact.
24. PRO→FREE downgrade does **not** trim existing executors (3 stay).
25. Now-over-limit FREE user still blocked from adding a 4th.
26. **Gap to confirm**: downgraded-but-over-limit owner's release still notifies/grants access to all 3 contacts — release path doesn't consult plan limits.
27. **Gap to confirm**: `PATCH /auth/me` may not actually block FREE users from setting `inactivityWindowMonths`, despite `plan-limits.ts` marking it disallowed — could be frontend-only enforcement.

### G. Contact removal mid-release

28–29. Owner removal / contact self-exit both blocked while a non-terminal release is active. 30. Removal succeeds once release is COMPLETED/CANCELLED; historical `ExecutorRelease` row remains. 31. Removed contact excluded from subsequent new releases only.

### H. Idempotency / resume

32. Report-generation failure leaves `Release` PENDING; next cron pass resumes (not duplicates).
33. COMPLETED release never gets a duplicate on re-run.
34. Concurrent `cancelRelease` calls are idempotent.
35. Admin cancelling an already-terminal release → `409`.

### I. Admin dev tooling itself (currently has no test coverage)

36. `set-status: NOTIFIED` defaults to 8-day backdate.
37. `set-status: RELEASING` cancels any pre-existing active release first.
38. Dev endpoints still require admin auth even with dev-tools enabled.

### J. Frontend/email touchpoints to check alongside

Dashboard status messaging, invite accept/decline UI, nudge email cadence/content, release-ready email (account vs. no-account variants), cancellation email, report-download page error states (not raw JSON), admin releases panel cancel/409 handling.

Two items (#26, #27) are flagged as likely real gaps rather than confirmed — worth a quick check before filing as bugs. Reset state between scenarios with the `/reset` dev endpoint to avoid cross-contamination.