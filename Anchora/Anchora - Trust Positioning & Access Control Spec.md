# Anchora — Trust Positioning & Access Control Spec

**Date:** 2026-09-04
**Context:** SEO teardown flagged "You, and only you" as architecturally false. Decision: maximum access controls, not zero-knowledge. Anchora CAN decrypt. Anchora MUST prove it won't except under defined conditions.
**Status:** Spec — ready for implementation

---

## Positioning Principle

> **Maximum-security with transparent access controls.** Not "zero-knowledge" (we can't), not "trust us" (we promise). "Here's exactly how we protect your data, who can access it, under what conditions, and here's the audit trail to prove it."

---

## 1. Copy: What Changes Where

### 1.1 Homepage — Security Section

**Remove:**
- ~~"You, and only you. Anchora employees can't access what's inside your vault."~~
- ~~"256-bit Encryption standard"~~
- ~~"100% Audit coverage"~~

**Replace with:**

```
## Security is not optional. It's the foundation.

You're trusting Anchora with your most sensitive financial information. We take
that responsibility seriously — from the moment you add a record to the moment
your trusted contact begins discovery.

### Encrypted everywhere
Every vault record is encrypted with AES-256-GCM at rest. Every connection is
protected by TLS 1.3 with HSTS enforced. Encryption keys are managed separately
from encrypted data and stored in dedicated hardware security infrastructure.

### No single person can access your data
Decrypting vault contents requires multi-party approval. No individual Anchora
employee, administrator, or founder can access your vault alone. Every access
is time-boxed, purpose-constrained, and irreversibly logged.

### You'll know if your data is ever accessed
If Anchora ever needs to access your vault data — for a release workflow,
compliance requirement, or support request you initiate — you will be notified
by email. Access is granted only for the specific purpose, for a limited
duration, and then immediately revoked.

### Independent verification
We publish our security architecture. We commit to regular independent
penetration testing. We maintain a responsible disclosure program at
security@anchora.com.ng.

### What you store stays yours
Anchora records where your assets are held. We never hold, move, or manage
your money. Your bank accounts, investments, crypto, and property stay exactly
where they are.

[Learn more about our security architecture](/security)
```

### 1.2 Security Page (`/security`) — Additions

Current page is good but needs:

**Replace the "Restricted vault access" section:**

```
### Access control architecture

Anchora uses a layered access model designed so no single person holds the
keys to your data.

**Key management**
Vault encryption keys are stored and managed in AWS Key Management Service
(KMS). Keys never appear in application code, configuration files, or
environment variables. Access to KMS operations is restricted to service
accounts — no human has direct KMS access.

**Multi-party approval (coming Q4 2026)**
Decrypting vault data for any purpose — release workflow, compliance, or user-
initiated support — will require approval from two authorised Anchora officers.
This is enforced at the key-management layer, not as a policy that can be
overridden. Until multi-party approval ships, vault access is restricted to the
automated release pipeline only.

**Purpose-bound, time-limited access**
When vault data is decrypted (e.g. during a release), the decryption
capability is granted for that specific purpose only and auto-expires. Keys
are rotated following each release.

**Owner notification**
If your vault data is decrypted by Anchora for any reason, you will be
notified by email within 24 hours, including the purpose, scope, and
duration of the access.

**Immutable audit trail**
Every access to vault data — including which records were decrypted, by what
process, when, and why — is written to an append-only audit log. Logs are
retained for a minimum of three years. You can request your account's access
log by emailing privacy@anchora.com.ng.
```

**Add new section: "What 'restricted access' means in practice"**

| Who | What they can see | What they cannot see | Condition |
|-----|-------------------|---------------------|-----------|
| You (account owner) | Everything you entered | Nothing | Authenticated session |
| Your Trusted Contact | Vault contents (post-release) | Nothing (same as you) | Identity verified + release triggered |
| Anchora automated systems | Asset categories, institution names, activity status | Account numbers, notes, documents, values | Normal operation |
| Anchora staff (release support) | Asset categories, guidance steps | Account numbers, notes, documents, values | During active release only |
| Anchora staff (escalation) | Vault contents | Nothing | Multi-party approval + time-boxed + owner notified |
| Anchora staff (any other time) | Nothing inside your vault | Everything inside your vault | Always |

### 1.3 Homepage — Other Copy Fixes

**Remove:**
- ~~"No legal battles"~~ → Replace with: "Clear guidance, no guessing"
- ~~"Verified Release model"~~ (undefined term) → Replace with: "Staged, verified release process"
- ~~"100% Audit coverage"~~ → Replace with: "Complete audit trail"

**Remove the "human review at every stage" claim entirely** or qualify as:
"Human review of the release process — not of your vault contents" (if true post-access-controls)

**Add FAQ entries to homepage:**
```
Q: Can Anchora see what's in my vault?
A: Not in normal operation. Your account numbers, notes, documents, and values
are encrypted. Anchora can only decrypt them under strict, multi-party
controlled, fully audited, and owner-notified conditions — specifically for
the release workflow you've configured.
```

### 1.4 Privacy Policy Updates

**Urgent — legal counsel should review. Minimum changes:**

1. **NDPR → NDPA:** Replace "Nigeria Data Protection Regulation (NDPR) 2019" with "Nigeria Data Protection Act 2023 (NDPA)" and "NDPB" with "NDPC" (Nigeria Data Protection Commission)
2. **"Staff cannot read plaintext contents"** → Replace with specific access-control language matching the security page
3. **Subprocessor disclosure:** List AWS, Cloudflare, Railway, Paystack, QoreID, email provider. State what data each processes.
4. **Business continuity:** Add: "If Anchora ceases operations, you will be notified and given 30 days to export your data before deletion."
5. **Data residency:** Add: "Vault data is hosted on AWS infrastructure in [region]. Document files are stored in Cloudflare R2."

---

## 2. Product Changes Required

### 2.1 Now — Copy-Only Changes (No Code)

These make trust claims accurate immediately:

| Surface | Change |
|---------|--------|
| Homepage security section | Replace per 1.1 above |
| Homepage "You, and only you" | Remove or qualify |
| Homepage "100% Audit coverage" | "Complete audit trail" |
| Homepage "No legal battles" | "Clear guidance, no guessing" |
| Security page "Restricted vault access" | Replace per 1.2 above |
| Security page "What access means" table | New section per 1.2 above |
| Privacy policy NDPR → NDPA | Update with legal counsel |
| Privacy policy subprocessors | List them |
| Footer / terms | Add business continuity commitment |
| robots.txt | Decide per-crawler (GPTBot, ClaudeBot, etc.) — document the decision |

### 2.2 Before Launching Content Marketing — Access Control Features

These make the claims systemically true, not just procedurally:

| Feature | What it does | Effort |
|---------|-------------|--------|
| **Multi-party approval on KMS** | AWS KMS key policy requiring 2 MFA-authenticated principals to use the key. Can use AWS KMS multi-party approval or a simple workflow: admin A requests → admin B approves → tokens combined → KMS decrypt allowed. | Medium (3-5 days) |
| **Time-boxed KMS grants** | CreateGrant with `Constraints.EncryptionContextSubset` and auto-expiring via Lambda cleanup. Each decrypt operation gets a 1-hour grant, then auto-revoked. | Low (1 day) |
| **Owner notification on access** | After any vault decryption: `notificationService.notifyOwnerVaultAccessed(ownerId, {purpose, scope, timestamp, grantId})`. Email template. | Low (1 day) |
| **Audit log for KMS operations** | AWS CloudTrail already logs KMS operations. Export KMS decrypt events to the audit log table with user-facing descriptions. | Low (1 day) |
| **Access log endpoint** | `GET /v1/account/access-log` — returns a user-readable log of when their vault data was accessed, by whom, and why. Uses existing audit log + CloudTrail. | Low (1 day) |
| **Published security architecture doc** | `/security/architecture` page with key-management diagram, access-control table, subprocessor list, penetration-test timeline. Static page, no backend. | Low (1 day) |

### 2.3 Later — Nice-to-Have

| Feature | Why |
|---------|-----|
| HSM / AWS CloudHSM | Hardware-bound keys, not just KMS. Higher trust tier. |
| Penetration test | Independent verification of claims. Publish summary. |
| SOC 2 / ISO 27001 | Formal certification for B2B/partner trust. Long-term. |

---

## 3. Implementation Sequence

### Phase 1: Copy Fixes (This Week)
- [ ] Rewrite homepage security section per 1.1
- [ ] Fix homepage claims: remove "You, and only you", "100% Audit coverage", "No legal battles"
- [ ] Update security page per 1.2
- [ ] Update privacy policy NDPR → NDPA + subprocessors (legal review)
- [ ] Decide robots.txt AI crawler policy
- [ ] Fix meta keywords tag reference from NDPR → NDPA

**After this phase:** Every public claim is accurate to the current architecture. No "You, and only you." No undefined percentages.

### Phase 2: Access Controls (Next 2 Weeks)
- [ ] Implement multi-party KMS approval
- [ ] Implement time-boxed KMS grants
- [ ] Implement owner notification on vault access
- [ ] Implement KMS audit log → user-facing access log
- [ ] Implement `GET /v1/account/access-log`
- [ ] Publish `/security/architecture` page

**After this phase:** Claims are systemically enforced, not just procedurally stated. Multi-party, time-boxed, owner-notified.

### Phase 3: Independent Verification (Q4 2026)
- [ ] Commission penetration test
- [ ] Publish summary (not full report — summary with findings and remediation)
- [ ] Evaluate SOC 2 / ISO 27001 timeline

---

## 4. Robots.txt — AI Crawler Decision

Current: blocks GPTBot, Google-Extended, ClaudeBot, Applebot-Extended.

**Recommendation for trust positioning:** Allow AI crawlers. Anchora's entire growth strategy depends on AI-search discoverability. Blocking them contradicts the SEO/content flywheel.

Allow-list:
```
User-agent: GPTBot
Allow: /

User-agent: Google-Extended
Allow: /

User-agent: ClaudeBot
Allow: /

User-agent: Applebot-Extended
Allow: /
```

Only block if there's a specific copyright/privacy concern — and document the reason.

---

## 5. Trust Architecture Summary (For Marketing)

> **How Anchora protects your data**
>
> 1. **Encrypted everywhere.** AES-256-GCM at rest. TLS 1.3 in transit. Your data is encrypted before it touches disk.
> 2. **No single person holds the keys.** Decryption requires multi-party approval. Even our founders can't access your vault alone.
> 3. **Time-boxed, purpose-bound access.** When access is granted, it's for a specific reason, auto-expires, and is immediately revoked afterward.
> 4. **You'll know.** If your vault data is ever accessed, you're notified — what was accessed, why, and when.
> 5. **Everything is logged.** Complete, immutable audit trail. You can request your access log.
> 6. **We never hold your money.** Anchora is a record, not a custodian. Your assets stay where they are.

---

## 6. Open Decisions

1. **Multi-party approval implementation:** AWS KMS key policy with MFA, or simple approval workflow (admin A + admin B tokens)? KMS policy is stronger but requires AWS MFA setup for each admin.
2. **"Human review" language:** Keep "human review of release process" but remove "human review at every stage" — or qualify explicitly as process review only?
3. **Penetration test timing:** Commission in Q4 or wait until all access controls ship?
4. **Publish security architecture:** How much detail? Full key-management diagram with KMS configuration, or high-level only?