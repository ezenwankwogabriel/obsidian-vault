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

| Field                                                   | Why                                               |
| ------------------------------------------------------- | ------------------------------------------------- |
| Asset category (`bank_account`, `pension_portal`, etc.) | Needed to generate guidance instructions          |
| Institution name                                        | Needed to generate guidance instructions          |
| `recoverySteps` / guidance template                     | Static text, not user data — Anchora authors this |
| Activity timestamps, inactivity status                  | Operational — no sensitive content                |
| Trusted Contact email, verification status              | Operational — no sensitive content                |

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

| Risk                                     | Mitigation                                                                                        |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------- |
| Owner loses password AND recovery phrase | Irrecoverable. Must re-create vault. Clear warning during setup.                                  |
| Trusted Contact loses Share B email      | Owner can re-send Share B while active. After inactivity, recovery requires Owner.                |
| Anchora server compromise                | Ciphertext useless without MVK, which requires both shares.                                       |
| Malicious Anchora insider                | Share A alone is useless (Shamir threshold = 2).                                                  |
| Browser crypto limitations               | Web Crypto API (SubtleCrypto) is mature, available in all modern browsers. No WebAssembly needed. |

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

## Open Questions

1. **Recovery phrase UX:** 12-word BIP39 — acceptable for Nigerian professional audience? Or passphrase?
2. **Multi-Trusted-Contact with Shamir:** Current spec = 1-of-N. With zero-knowledge, each contact needs a unique share? Or all get same Share B?
3. **Document encryption performance:** Large PDFs (5-50MB title deeds) — encrypt client-side before upload. Acceptable latency?
4. **Web Crypto API browser support:** Test on Opera Mini, UC Browser (Nigerian mobile market). Fallback needed?
5. **Existing user migration:** Wait for critical mass or offer immediately?

---

## References

- [Web Crypto API (MDN)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
- [Shamir's Secret Sharing (Wikipedia)](https://en.wikipedia.org/wiki/Shamir%27s_secret_sharing)
- [BIP39 Mnemonic](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- Current: `anchora inactivity-release pipeline.md`
- Current: `Anchora.md` — Sprint 9 (QoreID), Sprint 10 (Admin Dashboard)