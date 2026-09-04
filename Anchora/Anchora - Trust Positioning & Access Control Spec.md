# Anchora — Trust Positioning & Access Control Spec

**Date:** 2026-09-04
**Context:** SEO teardown flagged "You, and only you" as architecturally false. Decision: KMS-enforced no-human-access. The automated release pipeline is the ONLY principal that can decrypt vault data. No human — not founders, not admins, not support staff — can read vault contents.
**Status:** Spec — ready for implementation

---

## Positioning Principle

> **No human at Anchora can read your vault. Period.** Not zero-knowledge (we can't claim the system can't decrypt). But we CAN claim — and enforce — that no human has access to the decryption keys. The KMS key policy grants decrypt to exactly ONE principal: the NestJS service role. Every human IAM principal is denied. This is enforced at AWS, not in application code.

---

## How It Works

```
KMS key policy:
    Effect: Allow
    Principal: {AWS: "arn:aws:iam::*:role/anchor-backend-service"}
    Action: kms:Decrypt
    — and that's it. No human IAM users, roles, or groups.

Any admin IAM user → kms:Decrypt → DENIED (not in key policy)
Any founder IAM user → kms:Decrypt → DENIED
Any support staff → kms:Decrypt → DENIED

Only the release pipeline → kms:Decrypt → ALLOWED (service role)
```

### What Prevents Human Backdoors

KMS alone can't stop an admin who can deploy code or SSH into prod. So we also enforce:

| Control | What it prevents |
|---------|-----------------|
| **No human SSH to prod** | Can't exec on the box that holds the service role |
| **CI/CD-only deployment** | Can't push arbitrary code that calls decrypt |
| **PR review required** | Every deploy gets a second set of eyes |
| **CloudTrail on KMS** | Every decrypt call is logged. Alert on any call outside release pipeline hours/patterns |
| **DB fields are ciphertext** | Even raw DB access shows encrypted blobs |

With these five controls + the KMS key policy: **no Anchora human can ever read vault contents.**

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

### No human at Anchora can read your vault
Our encryption keys live in AWS Key Management Service. The key policy grants
access to exactly one thing: the automated release pipeline you've configured.
No employee, founder, or administrator — including our engineering team — can
decrypt your vault contents. This is enforced at the infrastructure layer, not
in application code.

### Encrypted everywhere
Every vault record is encrypted with AES-256-GCM before it touches disk. Every
connection is protected by TLS 1.3 with HSTS enforced. Your data is encrypted
at rest, in transit, and at every point in between.

### Everything is logged. You can see.
Every time the release pipeline accesses your vault data, it's logged — what
was accessed, when, and why. The audit trail is immutable. You can request
your account's access log at any time.

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

### 1.2 Security Page (`/security`) — Complete Rewrite of "Restricted Vault Access"

**Replace the entire "Restricted vault access" section:**

```
### Access control architecture

Your vault data is encrypted with AES-256-GCM. The encryption keys are stored
in AWS Key Management Service (KMS). Here's what makes our access model
different from most platforms:

**Only the release pipeline holds the keys — not people.**

The KMS key policy grants `kms:Decrypt` to exactly ONE principal: the Anchora
backend service role. No IAM user, role, or group belonging to any Anchora
employee — including founders and administrators — appears in the key policy.

This means:

- No employee can decrypt your vault data. The key policy denies them.
- No administrator can decrypt your vault data. The key policy denies them.
- No founder can decrypt your vault data. The key policy denies them.
- Even if someone obtained a copy of our database, the vault fields are
  ciphertext — useless without the KMS key they cannot access.

**When your data IS decrypted — and only then.**

The automated release pipeline is the only code path that calls `kms:Decrypt`.
It runs exclusively during a release that YOU configured: after the inactivity
window you set has passed, after your trusted contact's identity has been
verified, and only for the specific vault being released.

**We prevent human backdoors.**

KMS alone can't stop someone who can deploy code. So we enforce:

- No human SSH access to production infrastructure
- CI/CD-only deployment with mandatory pull request review
- Every deployment is logged and reviewed
- CloudTrail monitors every KMS operation — any decrypt call outside the
  release pipeline triggers an alert

**You can verify.**

Every vault decryption is logged to an immutable audit trail. You can request
your account's access log at any time by emailing privacy@anchora.com.ng.

We also publish our full key-management architecture at
/security/architecture.
```

**Replace the "What 'restricted access' means in practice" table:**

| Who | Can read vault contents? | Condition |
|-----|------------------------|-----------|
| You (account owner) | ✅ Yes | Authenticated session |
| Your Trusted Contact | ✅ Yes | Release triggered + identity verified |
| Anchora release pipeline (automated) | ✅ Yes | Release triggered + only the vault being released |
| Anchora employees, admins, founders | ❌ No | Never. KMS key policy denies them. |
| Law enforcement / government | ❌ No | Same as above. We refer them to you. |
| Third parties | ❌ No | We never share vault data. |

**What Anchora CAN see without decrypting your vault:**

| Data | Accessible to Anchora? | Why |
|------|----------------------|-----|
| Asset categories (bank_account, pension, etc.) | ✅ Yes | Needed to generate guidance instructions |
| Institution names | ✅ Yes | Needed to generate guidance instructions |
| Activity timestamps, inactivity status | ✅ Yes | Needed for release workflow |
| Account numbers, identifiers | ❌ No | Encrypted. KMS key policy blocks human access. |
| Notes, supporting documents | ❌ No | Encrypted. KMS key policy blocks human access. |
| Estimated values | ❌ No | Encrypted. KMS key policy blocks human access. |

### 1.3 Homepage — Other Copy Fixes

**Remove:**
- ~~"No legal battles"~~ → "Clear guidance, no guessing"
- ~~"Verified Release model"~~ (undefined term) → "Staged, verified release process"
- ~~"100% Audit coverage"~~ → "Complete, immutable audit trail"
- ~~"human review at every stage"~~ → Remove entirely

**Add FAQ entries to homepage:**

```
Q: Can Anchora employees see what's in my vault?
A: No. Our encryption keys are managed in AWS KMS. The key policy only allows
the automated release pipeline to decrypt data — no employee, founder, or
administrator has access. This is enforced at the infrastructure layer.

Q: Then how does the release work?
A: When the inactivity window you've set passes and your trusted contact's
identity is verified, the automated pipeline decrypts the vault data for that
specific release. It runs without human involvement. Every access is logged
and auditable.
```

### 1.4 Privacy Policy Updates

**Urgent — legal counsel should review. Minimum changes:**

1. **NDPR → NDPA:** Replace "Nigeria Data Protection Regulation (NDPR) 2019" with "Nigeria Data Protection Act 2023 (NDPA)" and "NDPB" with "NDPC" (Nigeria Data Protection Commission)
2. **"Staff cannot read plaintext contents"** → Replace with the specific KMS key policy language from the security page
3. **Subprocessor disclosure:** List AWS, Cloudflare, Railway, Paystack, QoreID, email provider. State what data each processes.
4. **Business continuity:** Add: "If Anchora ceases operations, you will be notified and given 30 days to export your data before deletion."
5. **Data residency:** Add: "Vault data is hosted on AWS infrastructure in [region]. Document files are stored in Cloudflare R2."

---

## 2. Product Changes Required

### 2.1 Now — Copy-Only Changes (No Code)

| Surface | Change |
|---------|--------|
| Homepage security section | Replace per 1.1 above |
| Homepage "You, and only you" | Remove — replaced with specific KMS enforcement language |
| Homepage "100% Audit coverage" | "Complete, immutable audit trail" |
| Homepage "human review at every stage" | Remove entirely |
| Homepage "No legal battles" | "Clear guidance, no guessing" |
| Homepage FAQ section | Add "Can employees see my vault?" entry |
| Security page "Restricted vault access" | Replace per 1.2 above |
| Security page access table | Replace per 1.2 above |
| Privacy policy NDPR → NDPA | Update with legal counsel |
| Privacy policy subprocessors | List them |
| Footer / terms | Add business continuity commitment |
| robots.txt | Allow AI crawlers (GPTBot, ClaudeBot, Google-Extended, Applebot-Extended) |

### 2.2 Before Launching Content Marketing — Infrastructure Changes

| Feature | What it does | Effort |
|---------|-------------|--------|
| **KMS key policy: service-role-only** | Restrict the vault KMS key so ONLY the NestJS service role can call `kms:Decrypt`. Remove all human IAM principals. | Low (1 hour — Terraform/CloudFormation update) |
| **No human SSH to prod** | Disable SSH / Session Manager access for all human IAM users. CI/CD pipeline is the only deployer. Break-glass access via temporary credential (logged, alerted). | Medium (1 day — IAM + security group changes) |
| **CloudTrail alert on KMS decrypt** | CloudWatch alarm: any `kms:Decrypt` event triggers a notification. Baseline the expected release-pipeline pattern; alert on anomalies. | Low (1 day) |
| **`GET /v1/account/access-log`** | Endpoint returning a user-readable log of vault data access events: timestamp, purpose (release / compliance / never), records accessed. Pulls from audit log + CloudTrail. | Low (1 day) |
| **Published security architecture page** | `/security/architecture` — static page with KMS key policy diagram, access-control table, subprocessor list, deployment/pipeline description, penetration-test timeline. | Low (half day) |

### 2.3 Later — Nice-to-Have

| Feature | Why |
|---------|-----|
| HSM / AWS CloudHSM | Hardware-bound keys. Even stronger than KMS soft keys. |
| Penetration test | Independent verification. Publish summary (not full report). |
| SOC 2 / ISO 27001 | Formal certification for B2B/partner trust. Long-term. |

---

## 3. Implementation Sequence

### Phase 1: Copy Fixes (This Week)
- [ ] Rewrite homepage security section per 1.1
- [ ] Fix homepage claims: remove "You, and only you", "100% Audit coverage", "No legal battles", "human review at every stage"
- [ ] Add homepage FAQ (employee access question)
- [ ] Update security page per 1.2
- [ ] Update privacy policy NDPR → NDPA + subprocessors (legal review)
- [ ] Allow AI crawlers in robots.txt
- [ ] Fix meta keywords tag reference from NDPR → NDPA

**After this phase:** Every public claim is accurate. "No human at Anchora can read your vault" is stated and — once Phase 2 ships — enforced.

### Phase 2: Infrastructure Enforcement (Next 2 Weeks)
- [ ] Restrict KMS key policy to service role only (remove all human IAM)
- [ ] Disable human SSH / Session Manager to prod
- [ ] Configure CloudTrail decrypt alert
- [ ] Implement `GET /v1/account/access-log`
- [ ] Publish `/security/architecture` page

**After this phase:** The claim is systemically enforced. No human CAN read vault contents because KMS denies them.

### Phase 3: Independent Verification (Q4 2026)
- [ ] Commission penetration test
- [ ] Publish summary (findings + remediation, not full report)
- [ ] Evaluate SOC 2 / ISO 27001 timeline

---

## 4. Robots.txt — AI Crawler Decision

Current: blocks GPTBot, Google-Extended, ClaudeBot, Applebot-Extended.

**Decision:** Allow all. Anchora's growth depends on AI-search discoverability. Blocking contradicts the content flywheel.

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

Document the reason if any crawler remains blocked (copyright/privacy concern).

---

## 5. Trust Architecture Summary (For Marketing)

> **How Anchora protects your data**
>
> 1. **No human can read your vault.** Encryption keys live in AWS KMS. The key policy only allows the automated release pipeline — no employee, founder, or administrator has access.
> 2. **Encrypted everywhere.** AES-256-GCM at rest. TLS 1.3 in transit. Your data is encrypted before it touches disk.
> 3. **Only the release you configured can access it.** The automated pipeline runs only when your inactivity window passes and your trusted contact is verified.
> 4. **Everything is logged.** Complete, immutable audit trail. You can request your access log.
> 5. **No human backdoors.** No SSH to production. CI/CD-only deployment. Mandatory code review. CloudTrail monitoring.
> 6. **We never hold your money.** Anchora is a record, not a custodian. Your assets stay where they are.

---

## 6. Open Decisions

1. **Break-glass access:** Should there be an emergency human recovery path (e.g. legal order, account recovery)? If yes, it requires a temporary KMS key policy update + multi-admin approval + forced owner notification. If no, accept that some data loss scenarios are unrecoverable. **Recommendation:** No break-glass for now. Revisit if operational necessity emerges.
2. **"Human review" language:** The scoped-down "human review of release process" is still misleading next to "no human can read your vault." **Recommendation:** Remove entirely from homepage. Only mention on security page under release verification (QoreID review, not content review).
3. **Penetration test timing:** Q4 2026 after all access controls ship, or earlier to validate claims before content marketing push?
4. **Security architecture page detail:** Full KMS key policy snippet + IAM diagram, or high-level architectural description only? **Recommendation:** High-level with a "request full architecture document" option for enterprise/partner evaluation.