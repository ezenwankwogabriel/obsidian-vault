# Anchora — Multi-Trusted Contact Implementation Spec

**Supersedes:** "primary + backup" model previously listed in roadmap
**Decision date:** Aug 3, 2026
**Status:** Pending implementation (post Sprint 9)

---

## 1. The Model

| | Free | Premium |
|---|---|---|
| **Max contacts** | 1 | Up to 5 |
| **Release logic** | 1-of-1 → receives inventory | N-of-N → all receive inventory simultaneously |
| **Verification** | QoreID identity verification | Same, per contact |

**Key principle:** No primary/backup, no voting, no M-of-N, no race. When release triggers, **all** verified trusted contacts receive the asset inventory simultaneously. One-way broadcast: every contact gets the same information at the same time. Simplicity — no sequential activation, no "first to verify" gate.

---

## 2. Data Model Changes

### New Table: `trusted_contacts`

```sql
CREATE TABLE trusted_contacts (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  owner_id      UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
  name          TEXT NOT NULL,
  email         TEXT NOT NULL,
  phone         TEXT,
  relationship  TEXT,                        -- "spouse", "sibling", "lawyer", etc.
  
  -- Invitation & account linking
  invite_status  TEXT NOT NULL DEFAULT 'pending'
                  CHECK (invite_status IN ('pending', 'accepted', 'verified', 'removed')),
  invite_token   TEXT UNIQUE,                -- one-time use, for the invite link
  invited_at     TIMESTAMPTZ,
  accepted_at    TIMESTAMPTZ,
  verified_at    TIMESTAMPTZ,                -- QoreID verification completed
  linked_user_id UUID REFERENCES users(id) ON DELETE SET NULL,  -- if contact creates their own account
  
  -- Audit
  created_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  
  -- Constraints
  CONSTRAINT unique_active_email_per_owner UNIQUE (owner_id, email)
    DEFERRABLE INITIALLY DEFERRED,           -- allows re-invite after removal
  CONSTRAINT max_premium_contacts CHECK (
    -- Enforced via app logic, not DB constraint, due to tier awareness
    TRUE
  )
);

CREATE INDEX idx_trusted_contacts_owner ON trusted_contacts(owner_id);
CREATE INDEX idx_trusted_contacts_invite_token ON trusted_contacts(invite_token)
  WHERE invite_status = 'pending';
```

### Changes to Existing Tables

**`accounts` table (or wherever plan info lives):**
- Add `plan_tier` column if not already present: `'free' | 'premium'`
- The contact limit is derived from tier, not a separate column

**No changes needed to:**
- `vault_records` or `assets` — contacts don't own records, they just get guided discovery access
- The release/trigger state machine — the trigger logic (inactivity detection, warnings) stays the same

---

## 3. Invite and Onboarding Flow

### Owner Side

**Step 1: Add a contact**
- Owner fills: name, email, phone (optional), relationship label
- Free tier: only one contact slot. Contact management page shows "Add contact" button (hidden when slot full, or disabled with upgrade prompt)
- Premium tier: up to 5 slots. Add/remove freely while under the limit

**Step 2: Invite is sent**
- Invite link generated (`POST /api/trusted-contacts` → returns `invite_token`)
- Email sent (or SMS, if implemented) with link: `https://app.anchora.com.ng/invite?token=<token>`
- Contact shown in list with status `pending`

**Step 3: Manage contacts**
- Owner sees a list of all contacts + their current status
- Can remove any contact at any time (removed = they lose access, invite invalidated)

### Contact Side

**Step 1: Receive invite**
- Landing page: "Gabriel has designated you as a trusted contact on Anchora. Anchora helps people document their financial and physical assets so someone they trust can find them when needed."
- Two buttons: **Accept** | **Learn More**

**Step 2: Accept**
- If contact already has an Anchora account → link to existing account (prompt: "You already have an account. Link this invitation?")
- If new → create lightweight account (name, email, phone, password). No vault, no billing — just a contact profile
- After acceptance → status changes to `accepted`

**Step 3: Verify identity (when release triggers)**
- Identity verification (QoreID) happens when the release process actually triggers
- Not required during invite acceptance — just account linking
- Verified status is set only after successful QoreID during the release flow

### UI States for Contact List

| Status | Badge | Meaning |
|--------|-------|---------|
| `pending` | Grey "Invited" | Invite sent, not yet accepted |
| `accepted` | Blue "Linked" | Has an account, linked to this vault |
| `verified` | Green "Verified" | Passed QoreID (release context) |
| `removed` | Red strikethrough | Owner removed them (historical only) |

---

## 4. Release Flow (Broadcast — N-of-N)

### Trigger (unchanged)
- Inactivity timer expires (3–18 months, set by owner)
- Warning emails already sent per existing flow
- Grace period expires → release is triggered

### What Changes: Multi-Contact Broadcast

**All active contacts receive the inventory simultaneously:**
- Email to each contact with status `accepted` or `verified`
- Message: "Gabriel has been inactive for [X months]. Here is the asset inventory they asked you to have."
- Each receives an identical guided-discovery packet: asset categories + institutional contact info
- No "first to verify" gate — everyone gets the same information at the same time

**Verification happens before release, not as a race:**
- Contacts verify (QoreID) during onboarding, not during release
- Only contacts who completed verification before the trigger receive the inventory
- Unverified contacts get a notification: "[Owner] has been inactive. Verify your identity to access their inventory."
- Once they verify, the inventory is available to them (no time limit)

**Owner re-activates:**
- Owner logs in or performs any activity → release is cancelled immediately
- All contacts lose access
- Standard "you're back" notification sent to owner and contacts

### Why Broadcast Is Safe for Anchora
- Contacts see *where assets are held*, not credentials
- No asset transfer, no monetary loss risk
- Multiple eyes = more chance someone actually follows through
- Audit trail records every access
- No primary/backup hierarchy means no single point of failure

---

## 5. API Endpoints

| Method | Endpoint | Purpose | Auth |
|--------|----------|---------|------|
| `GET` | `/api/trusted-contacts` | List owner's trusted contacts | Owner JWT |
| `POST` | `/api/trusted-contacts` | Add a new contact + send invite | Owner JWT |
| `DELETE` | `/api/trusted-contacts/:id` | Remove a contact | Owner JWT |
| `PATCH` | `/api/trusted-contacts/:id` | Edit name/phone/relationship | Owner JWT |
| `GET` | `/api/invite/:token` | Resolve invite token (landing page) | None |
| `POST` | `/api/invite/:token/accept` | Accept invitation | Contact JWT (or create account) |
| `GET` | `/api/release/contacts` | List verified contacts who accessed | Internal/admin |

---

## 6. UI Changes

### Vault Dashboard — Contact Section

Current: Shows one contact card with name + status.
New: Shows a **contact list** with:
- Contact cards in a vertical list (name, email, relationship, status badge)
- "Add contact" button (Free: disabled with upgrade nudge when slot full / Premium: always visible under 5)
- Each card has a "Remove" action

### Free Tier — Contact Limit UX

When the free user has 1 contact and tries to add another:

> **Inline prompt (not modal):**
> "You've reached the limit of 1 trusted contact on the Free plan. Upgrade to Premium to add up to 5 trusted contacts — so your vault is covered no matter what."
> [Compare plans] button → pricing page

### Premium — 5 Contact Slots

- Show a progress indicator: "3 of 5 trusted contacts added"
- "Add contact" button visible as long as count < 5
- Removing a contact frees the slot immediately

### Invite Management
- Contact card shows invite status
- "Resend invite" button for `pending` contacts (resends email with same token)
- No "unlimited resend" — cap at 3 resends to prevent abuse (optional, can skip MVP)

---

## 7. Edge Cases

### Owner Downgrades from Premium → Free

**Scenario:** User was Premium, has 4 contacts, then cancels/downgrades to Free.

**Handling:**
- No contacts are force-removed at cancellation
- At `currentPeriodEnd`, if owner still has >1 contact, they get a notice:
  > "Your Premium plan has ended. Please remove 3 trusted contacts to continue on Free. Until then, you can't add new contacts."
- The owner can manually remove down to 1. If they don't, the system keeps the contacts in `active` state but blocks adding new ones
- Grace period before locking? Not needed — just freeze adds

### Contact Changes Their Email

Owner edits the contact's email → generates a new invite token, resets status to `pending`, new invite sent. Old link invalidated.

### Contact Is Unresponsive

- No action needed. Other contacts proceed as normal
- Owner can remove and replace them at any time
- If all contacts are unresponsive... that's an edge case, but the same as today with 1 contact being unreachable

### Owner Removes a Contact During Release

- That contact's access is immediately revoked
- Other contacts are unaffected
- Audit log entry created

### Owner Dies Without Any Contact Having Verified

Same problem as today with 1 contact. No regression. Mitigation: Premium's 5 contacts makes it much more likely *someone* will follow through.

---

## 8. Implementation Order

### Phase 1: Data & Invite Flow (estimated: 2–3 days)
1. Create `trusted_contacts` table
2. Write API routes: add, list, remove, edit contacts
3. Build invite acceptance flow (landing page + account linking)
4. Migrate existing single-contact data to new table

### Phase 2: UI (estimated: 2–3 days)
5. Replace single-contact card with contact list on dashboard
6. Add contact form + validation
7. Free-tier upgrade nudge
8. Premium contact limit indicator

### Phase 3: Release Flow (estimated: 1–2 days)
9. Modify release trigger to notify all active contacts
10. Build "first-to-verify" notification logic
11. Handle subsequent verifications

### Phase 4: Polish (estimated: 1 day)
12. Resend invite
13. Downgrade edge case handling
14. Audit trail updates

---

## 9. Migration from Current Single-Contact Model

**Current state:** A single `trusted_contact_id` on the `accounts` table (or similar).

**Migration steps:**
1. Run `CREATE TABLE trusted_contacts` (above)
2. For each account with an existing trusted contact:
   ```sql
   INSERT INTO trusted_contacts (owner_id, name, email, ...)
   SELECT id, trusted_contact_name, trusted_contact_email, ...
   FROM accounts
   WHERE trusted_contact_id IS NOT NULL;
   ```
3. Drop the old column from `accounts`
4. Update all code referencing the old single-contact schema
5. Deploy API changes before UI changes (backward-compatible)

---

## 10. Risks and Mitigations

| Risk | Likelihood | Mitigation |
|------|-----------|------------|
| Contact creates duplicate accounts on multiple invites | Low | Email-based dedup in invites + link-to-existing-account prompt |
| Owner spam-invites people | Low | Cap at 5 for Premium, limit resends |
| Billing race: Free user bypasses contact limit via API | Low | Server-enforced limit, not just client-side |
| "Verified" contacts with no real relationship (spam) | Low | Owner chooses who to invite; QoreID is real identity check, not a social network |

---

## 11. Open Questions for Gabby

1. **Parallel access**: Should multiple verified contacts all get access simultaneously (current design), or first-to-verify and others get locked out?
2. **Invite resend limit**: 3 resends, or unlimited? (3 is fine for MVP)
3. **Owner edits contact**: Edit inline in the list, or remove-and-re-add?