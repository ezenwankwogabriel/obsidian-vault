# Anchora — Zero-Knowledge Architecture Spec

**Date:** 2026-09-04
**Status:** Decision — zero-knowledge for vault contents
**Context:** SEO teardown flagged "You, and only you" as architecturally false under server-held-keys. Decision: Anchora cannot read vault contents.

---

## Core Principle

> Anchora stores ciphertext. Only the Account Owner and their designated Trusted Contact(s) can decrypt vault contents. Anchora employees, infrastructure, and administrators have zero technical capability to read vault data.

---

## What Gets Encrypted (Zero-Knowledge)

| Field | Current | New |
|-------|---------|-----|
| `accountNumber` / `identifier` | Field-level AES-256 (server keys) | Client-side, vault-key-encrypted |
| `notes` | Field-level AES-256 (server keys) | Client-side, vault-key-encrypted |
| `estimatedValue` (kobo) | Field-level AES-256 (server keys) | Client-side, vault-key-encrypted |
| Supporting documents (PDF, images) | R2 storage | Client-side encrypted before upload |
| `assetName` (friendly label) | Plaintext | Plaintext (needed for category index) |

## What Stays Server-Readable

| Field | Why |
|-------|-----|
| Asset category (`bank_account`, `pension_portal`, etc.) | Needed to generate guidance instructions |
| Institution name | Needed to generate guidance instructions |
| `recoverySteps` / guidance template | Static text, not user data — Anchora authors this |
| Activity timestamps, inactivity status | Operational — no sensitive content |
| Trusted Contact email, verification status | Operational — no sensitive content |

**Result:** Anchora can generate "Contact GTBank estate desk. Bring letters of administration and valid ID." without knowing the account number or seeing attached documents.

---

## Key Architecture

### Vault Key Hierarchy

```
Master Vault Key (MVK)
    └── 256-bit symmetric key, generated client-side on vault creation
        Never leaves the client in plaintext

MVK encrypted with Owner Key
    └── Stored on Anchora servers
    └── Owner decrypts with their key (derived from auth + recovery phrase)

MVK encrypted with Release Key
    └── Split into 2+ shares (Shamir's Secret Sharing)
    └── Share 1: Stored with Anchora
    └── Share 2: Delivered to Trusted Contact at designation time
    └── Both shares needed to reconstruct MVK
    └── Anchora alone CANNOT reconstruct (one share is useless)

Recovery Phrase
    └── 12-word BIP39 mnemonic, shown once during vault creation
    └── Owner must store this offline
    └── Used to derive Owner Key if primary auth is lost
    └── Anchora never stores or sees the recovery phrase
```

### Key Flow: Vault Creation

```
1. Owner signs up → generates MVK client-side (Web Crypto API)
2. Owner derives Owner Key from password + salt (PBKDF2/Argon2)
3. MVK encrypted with Owner Key → stored on Anchora as `encryptedMvk`
4. Recovery Phrase generated → shown ONCE to Owner → never stored by Anchora
5. Release Key = Shamir-split(MVK, threshold=2, shares=2)
   - Share A → stored on Anchora
   - Share B → emailed to Trusted Contact as "vault access fragment"
6. All future asset data encrypted with MVK before upload
```

### Key Flow: Trusted Contact Release

```
1. Release triggered (inactivity window passed, verification complete)
2. Anchora releases Share A to Trusted Contact
3. Trusted Contact already has Share B (from designation email)
4. Client-side Shamir reconstruction → recovers MVK
5. MVK decrypts vault contents client-side in browser
6. Trusted Contact sees asset names, identifiers, documents, guidance steps
```

### Key Flow: Owner Access (Normal Use)

```
1. Owner logs in → decrypts `encryptedMvk` with Owner Key
2. MVK held in memory (never persisted to disk, never sent to server)
3. All vault reads: server returns ciphertext → client decrypts with MVK
4. All vault writes: client encrypts with MVK → uploads ciphertext
```

---

## What This Changes

### Backend (NestJS)

| Module | Change |
|--------|--------|
| **Encryption service** | Remove server-side encryption of vault field data. Store only ciphertext + `encryptedMvk` + Release Key Share A. |
| **Vault CRUD** | `POST/PATCH` vault records accept pre-encrypted fields. No server-side crypto on field data. |
| **Report generation** | REMOVED. No server-side report (Puppeteer/KMS pipeline). Replaced by client-side decryption + display. |
| **Admin dashboard** | Can see metadata (category, institution, timestamps). Cannot see identifiers, notes, documents, values. |
| **Release service** | Remove report generation. Add Release Key Share A delivery endpoint. |
| **KMS integration** | Potentially removed/reduced (no server-side data decryption needed). |

### Frontend (Next.js)

| Module | Change |
|--------|--------|
| **Vault creation flow** | New step: generate MVK, show recovery phrase, encrypt. |
| **Vault read/write** | All field values encrypted/decrypted client-side via Web Crypto API. |
| **Document upload** | Encrypt file client-side before R2 upload. |
| **Trusted Contact onboarding** | Receive Share B email. Store securely. |
| **Guided discovery UI** | Client-side decrypt + render. Replaces PDF report download. |
| **Key management** | MVK in memory, encrypted MVK in localStorage/sessionStorage. Recovery phrase view once then clear. |

### What Breaks

| Feature | Status | Replacement |
|---------|--------|-------------|
| Server-side PDF report generation | Removed | Client-side rendered discovery view |
| "Human review at every stage" | Content-level review removed | Process-level review only (verification, timing, notifications) |
| Admin ability to view vault contents | Removed | Admin sees metadata only |
| `estimatedValue` aggregation/trends | Removed (server can't read values) | N/A — never built anyway |
| Document preview in admin | Removed | N/A |

### What Gets Stronger

| Claim | Before | After |
|-------|--------|-------|
| "You, and only you" | Architecturally false | Architecturally true |
| "Anchora cannot access your data" | Misleading | Verifiable |
| Security page | Vague "256-bit encryption" | Specific: "Client-side AES-256-GCM. Anchora holds ciphertext only. Key escrow via Shamir's Secret Sharing." |
| Trust differentiation | Claims only | Provable architecture |

---

## Trusted Contact Experience (After Release)

### Current: PDF Report
- Anchora generates PDF with Puppeteer
- Attaches supporting docs
- Trusted Contact downloads

### New: Client-Side Discovery View
- Trusted Contact enters Share A + Share B in browser
- Client-side Shamir reconstructs MVK
- MVK decrypts vault data locally
- Rendered as structured discovery view (same information, no PDF generation)
- Each asset shows: identifier, notes, attached documents (decrypted in browser), guidance steps

---

## Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| Owner loses password AND recovery phrase | Irrecoverable. Must re-create vault. Clear warning during setup. |
| Trusted Contact loses Share B email | Owner can re-send Share B while active. After inactivity, recovery requires Owner. |
| Anchora server compromise | Ciphertext useless without MVK, which requires both shares. |
| Malicious Anchora insider | Share A alone is useless (Shamir threshold = 2). |
| Browser crypto limitations | Web Crypto API (SubtleCrypto) is mature, available in all modern browsers. No WebAssembly needed. |

---

## Implementation Phases

### Phase 1: Zero-Knowledge for New Vaults
- New vault creation flow with client-side key generation
- New vault CRUD with client-side encryption
- Backend removes server-side field encryption for new records
- Existing vaults remain on server-encrypted model (migration later)

### Phase 2: Client-Side Discovery
- Trusted Contact discovery UI (client-side decrypt)
- Remove server-side report generation for new releases
- Release Key Share delivery flow

### Phase 3: Migration (Existing Users)
- Tool to re-encrypt existing vault data client-side
- User-initiated (login → decrypt old → re-encrypt with MVK → upload)
- Old server-encrypted data deleted after migration confirmed

---

## SEO / Trust Messaging

Current claims to fix:
- ~~"256-bit Encryption standard"~~ → "Client-side AES-256-GCM encryption. Anchora never has access to your vault contents."
- ~~"You, and only you"~~ → Now architecturally true. Keep it.
- ~~"100% Audit coverage"~~ → Replace with: "Every action is logged. Vault contents are end-to-end encrypted — even Anchora cannot read them."
- ~~"human review at every stage"~~ → "Human review of every release trigger and identity verification. Vault contents are never accessible to Anchora staff."

New claims unlocked:
- "Zero-knowledge architecture"
- "We literally cannot read your data"
- "If our servers are compromised, your data remains encrypted"
- Published architecture document (trust via transparency)

---

## Passkey-Based Alternative (WebAuthn PRF)

### How It Works

Passkeys can do more than authentication. Via the **WebAuthn PRF extension**, a passkey can derive a deterministic symmetric key:

```
Owner creates vault → registers passkey → PRF derives Vault Encryption Key (VEK)
    ↓
VEK never stored anywhere. Regenerated in-browser each time owner authenticates with passkey.
    ↓
No password. No recovery phrase. Hardware-backed (Secure Enclave / TPM).
```

This is what Bitwarden and 1Password use for their zero-knowledge vaults.

### Trusted Contact Flow (No Shamir Needed)

```
Owner designates TC → TC must register a passkey (requires notification — see below)
    ↓
Owner's browser: encrypts VEK with TC's passkey public key (hybrid encryption)
    ↓
Encrypted VEK stored on Anchora. Anchora cannot decrypt (no private key).
    ↓
Release triggered → Anchora delivers encrypted VEK to TC
    ↓
TC authenticates with passkey → browser decrypts VEK → accesses vault
```

**Advantage over Shamir:** No "Share B" to email, store, or lose. The TC's face/fingerprint IS the key. Each TC gets their own encrypted copy of VEK — natural fit for multi-TC.

**Still requires notification at designation** — TC must register a passkey to receive the encrypted VEK.

### Comparison

| | Password + Shamir (v1) | Passkey + PRF (alternative) |
|---|---|---|
| Owner key derivation | Password → PBKDF2 → key | Passkey PRF → key (hardware-backed) |
| Recovery | 12-word phrase | Passkey synced to iCloud/Google (multi-device) |
| TC key delivery | Share B emailed | TC registers passkey → encrypted VEK stored server-side |
| Phishing resistance | Password-phishable | Passkey cannot be phished |
| Nigerian mobile | Works on any browser | Requires platform passkey support (growing, not universal) |
| UX for owner | Remember password | Biometric (Face ID / fingerprint) |
| UX for TC | Store a fragment | Register once, biometric thereafter |
| Implementation complexity | Medium (Shamir, key derivation) | Lower (WebAuthn API handles crypto) |

### Recommendation

Passkey + PRF is the better architecture if Nigerian target devices support it. Simpler implementation, stronger security, better UX. Fall back to password + Shamir for devices without passkey support.

---

## The TC Notification Problem

### Current Behavior

Anchora does **not** notify Trusted Contacts at designation. The rationale: avoid alarming someone with "you've been designated as someone's death-trigger contact."

### Why This Conflicts with Zero-Knowledge

You cannot have zero-knowledge AND a silent contact. At some point, the TC must hold or be able to regenerate a decryption key:

| Approach | When TC gets the key | Requires notification? |
|----------|---------------------|----------------------|
| Shamir Share B | At designation (emailed) | Yes |
| Passkey registration | At designation (register) | Yes |
| Delayed delivery at warning | When inactivity warning fires | Notification still happens |
| Anchora holds full key | Never (TC gets it from Anchora at release) | No — but breaks zero-knowledge |

**There is no architectural workaround.** Either Anchora holds the key (not zero-knowledge) or the TC must receive/generate a key at some point (requires contact).

### Options

| Option | Notification Timing | Message | Risk |
|--------|-------------------|---------|------|
| **A: Designate-and-notify** | Immediately | "Gabby has added you as a trusted person on Anchora. Store this access key — you'll only need it if they're unable to manage their affairs." | Minimal. Framing matters. Not "you're their death contact." |
| **B: Silent designate, notify on warning** | When inactivity warning fires | "Gabby designated you as a trusted person. They haven't been active recently. Here's your access key." | High. The TC's email could be dead by then. Or TC has changed. Or the email goes to spam. |
| **C: Passkey at designation, silent until release** | Immediately for registration, then silent | "Gabby has added you to Anchora. Please set up your secure access — it takes 30 seconds and you won't need it unless something happens." | Low. Passkey registration is a one-time, non-alarming action. No "key" to store or lose. |

### Recommendation: Option C

- **Notify at designation**, but frame it as a routine setup action, not a death trigger
- Language: "Register your access" not "You're their beneficiary"
- Single passkey registration. Done. Silent for months/years after
- If the TC ignores it, re-prompt periodically (not tied to inactivity — just "complete your setup")

---

## Open Questions

1. **Passkey support in Nigerian mobile browsers:** Chrome Android has full support. iOS Safari has full support. Opera Mini does NOT support WebAuthn. UC Browser — uncertain. Need to test. What's the fallback for unsupported browsers?
2. **Passkey vs password + Shamir — which path?** Passkey is simpler and stronger but requires platform support. Shamir works everywhere but has worse UX (secrets to store).
3. **TC notification at designation — acceptable?** Changes current "silent" policy. Worth it for zero-knowledge?
4. **Document encryption performance:** Large PDFs (5-50MB title deeds) — encrypt client-side. Acceptable latency?
5. **Existing user migration:** Wait for critical mass or offer immediately?

---

## References

- [Web Crypto API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- [Shamir's Secret Sharing (Wikipedia)](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)
- [BIP39 Mnemonic](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- Current: `anchora inactivity-release pipeline.md`
- Current: `Anchora.md` — Sprint 9 (QoreID), Sprint 10 (Admin Dashboard)