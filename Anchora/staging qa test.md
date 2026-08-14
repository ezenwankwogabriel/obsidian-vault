**Prerequisites**

- Backend: `DEV_TOOLS_ENABLED=true` (or non-prod `NODE_ENV`) on staging.
- Frontend: `NEXT_PUBLIC_DEV_TOOLS=true` set at **build time** for the staging frontend — separate flag, `src/app/admin/dev-tools/page.tsx` renders `null` without it, even if you're a logged-in admin.
- Two browser sessions needed side by side (owner cookie `anchora_auth`, admin cookie `anchora_admin_auth`, and executor logins are just a different `anchora_auth` session) — easiest is one normal window + one incognito window, since owner and executor are different `User` accounts and can't share a session.

Passwords (from the seed script): owners → `OwnerQA1234!`, executors → `ExecQA1234!`, admin → `AdminQA1234!`, readonly admin → `ReadOnlyQA1234!`.

run in cli
```
railway login
railway link
railway run bash -c "RELQA_BASE_EMAIL=anchora@gmail.com npm run db:seed:release-qa"
```

---

### Admin session (one tab, stays logged in throughout)

1. `/admin/login` → `anchora+relqa-admin@gmail.com` / `AdminQA1234!`.
2. `/admin/dev-tools` — this is your control panel for every timing shortcut. Paste an owner's email into "User ID or email address" → **Load** → the state card + action controls appear.

### O1 `relqa-owner-solo` — happy path, check-in cancel, idempotency (A1, C10-13, H32-35)

1. Admin tab: load `anchora+relqa-owner-solo@gmail.com` → Stage: `NOTIFIED`, backdate `8` → **Set status** → **Run inactivity check** → state card should flip to `RELEASING` with an active release link.
2. Incognito tab: `/login` as `anchora+relqa-exec-solo@gmail.com` → `/estates` → the pending invite from the first-contact auto-notify appears → **Accept**.
3. Normal tab: `/login` as `anchora+relqa-owner-solo@gmail.com` → `/dashboard` (any authenticated page load triggers the check-in). Back in admin tab, reload state → release should now show `CANCELLED`, status back to `ACTIVE`.
4. Repeat step 1 to re-release, then in admin tab click **Reset user** to clean up between passes — note it pops a `window.confirm()` dialog, so it's not accidental-click-proof, but do read it before confirming.

[observation: on reset/owner login, release is canceled, but the trusted contact still had access (being fixed now)]

### O2 `relqa-owner-multi` — grace/nudge/decline/equal-access, **and the suspected bug** (B4-9, D14-16)

1. Admin tab: load `anchora+relqa-owner-multi@gmail.com` → Stage `NOTIFIED`, backdate `3` → **Set status** → **Run inactivity check** → confirm it stays `NOTIFIED` (still in grace).
2. Set status again, backdate `10` → **Run inactivity check** → nudge should fire to the two pending contacts (check Resend dashboard, or the executors' Gmail inboxes via the `+relqa-exec-declined`/`+relqa-exec-accepted` tags).
3. **Bug check**: incognito tab → `/login` as `anchora+relqa-exec-declined@gmail.com` → `/estates` → **Decline** on the pending invite. Immediately reload the admin state card for O2 — does `inactivityStatus` stay `NOTIFIED` (correct, per the cron's "everyone must decline" rule) or does it jump straight to `ACTIVE` (the bug in `estates.service.ts::declineInvite`, which resets on any single decline)? This is the one to screenshot either way.
4. Third tab (or reuse incognito after logging out): `/login` as `anchora+relqa-exec-accepted@gmail.com` → `/estates` → **Accept** → admin tab **Run inactivity check** → should advance straight to `RELEASING` even though `relqa-exec-never-notified` never responded (equal access).
5. As the accepted contact, `/estates` → open the now-active estate → confirm report access works while `relqa-exec-declined` correctly cannot access it (403 if you try their session against the same release via the API — there's no UI path for a declined contact to attempt this, which itself is worth noting).

### O3 `relqa-owner-empty` — zero contacts, FREE limit (B9, F23)

1. Admin tab: load, Stage `NOTIFIED` backdate `8` → **Run inactivity check** → should reset to `ACTIVE` immediately (no contacts).
2. Normal tab: `/login` as `anchora+relqa-owner-empty@gmail.com` → `/executor` → **Add trusted contact** once (succeeds) → try adding a 2nd → should be blocked (FREE limit = 1). Note this owner starts with 0 seeded executors, so you'll add the first one live here.

### O4 `relqa-owner-downgrade` — plan downgrade doesn't trim executors (F24-27)

1. Normal tab: `/login` as `anchora+relqa-owner-downgrade@gmail.com` → `/executor` → confirm all 3 pre-accepted contacts are listed, "Add another" is disabled/blocked (PRO limit=3, already at cap).
2. Admin tab: load this owner → Plan select `FREE` → **Set plan**.
3. Back in normal tab, refresh `/executor` → all 3 should still be listed despite now being FREE (limit=1) → try adding a 4th → blocked.
4. Admin tab: Stage `NOTIFIED` backdate `8` → **Run inactivity check** → confirm `RELEASING` still creates `ExecutorRelease` rows / notifies all 3, not just 1.
5. `inactivityWindowMonths` gap check needs the API directly (no UI field for it that I found) — `PATCH /api/v1/auth/me {"inactivityWindowMonths": 6}` while this owner is FREE, check whether it 200s or 403s.

### O5 `relqa-owner-verify` — identity-verification gating (E17-22)

Don't use the Settings → Identity Verification UI here — it requires a real NIN + live selfie against Dojah, which fake seeded people can't pass. Verification statuses are already pre-set directly in the DB (`unverified`/`failed`/`verified`).

1. Admin tab: load owner → Stage `RELEASING` → **Set status** (skips grace entirely, creates the release immediately).
2. Incognito, log in as each executor in turn and go to `/estates` → open the estate:
    - `relqa-exec-unverified` → drawer shows "Action required" / "Verify your identity" prompt, no download.
    - `relqa-exec-failed` → same prompt (this is the one worth double-checking renders the same as unverified, not a distinct error state).
    - `relqa-exec-verified` → drawer shows "Report available" with a working **Download** button.
3. For the 30-day window / 24h signed-URL checks, those need a direct DB timestamp edit on the `Release`/`ExecutorRelease` row (no UI or dev-tool path) — not something to click through.

### O6 `relqa-owner-removal` — removal blocked mid-release (G28-31)

1. Admin tab: load owner → Stage `RELEASING` → **Set status**.
2. Normal tab: `/login` as `anchora+relqa-owner-removal@gmail.com` → `/executor` → try **Remove trusted contact** on either card → should be blocked while release is non-terminal.
3. Admin tab → **Reset user** (cancels the release, terminal state) → back in `/executor`, removal should now succeed.
4. Admin tab → Stage `RELEASING` again → confirm the removed contact is excluded from the new release, the remaining one is included.

### Read-only admin check (I38)

`/admin/login` as `anchora+relqa-readonly@gmail.com` → try to reach `/admin/dev-tools` and use any action → should be rejected (403) even though the page itself may render, since the guard is role-based server-side, not just page-visibility.

---

Two things with no UI/dev-tool path at all, confirmed only reachable via direct SQL or raw API calls (already noted in the seed-script plan): backdating `lastActivityAt`/`lastReminderSentAt` for the organic ACTIVE→NOTIFIED test, and backdating a `Release.triggeredAt` for the 30-day window test.