# Anchora — Zero-Knowledge Architecture (Final)

**Date:** 2026-09-04
**Context:** We need zero-knowledge encryption for Anchora's web MVP. The Trusted Contact may not accept their invite immediately. Industry-standard, maintainable, no reinvention.
**Reference:** Bitwarden Emergency Access model + Web Crypto API
**Status:** Spec — ready for implementation

---

## Root Problem

Owner stores vault data. Anchora cannot read it. When the owner goes inactive for their configured window, the Trusted Contact must be able to decrypt and access the vault — even if the TC hasn't accepted their invitation yet.

---

## Recommended Architecture: Bitwarden Model + Recovery Passphrase

### Why This Model

- **Bitwarden Emergency Access** is the industry-standard solution for exactly this use case (zero-knowledge handoff to a trusted party). It powers millions of users.
- **Web Crypto API** provides every primitive we need — natively, with zero dependencies, hardware-accelerated, in every browser. No WASM. No libraries.
- **Recovery passphrase** covers the "TC hasn't accepted yet" edge case. Simple. Not complex. Proven.

### Cryptographic Stack (Zero Dependencies)

| Primitive | Algorithm | Standard | Why |
|-----------|-----------|----------|-----|
| Key derivation | PBKDF2-SHA256, 600,000 iterations | Web Crypto API | Bitwarden 2026 default. OWASP recommended. |
| Symmetric encryption | AES-256-GCM | Web Crypto API | Authenticated encryption. Hardware-accelerated. |
| Asymmetric (TC wrap) | ECDH-P256 + AES-256-GCM | Web Crypto API | Cleaner than RSA-OAEP. Same as modern E2E apps. |
| Random bytes | `crypto.getRandomValues()` | Web Crypto API | CSPRNG, hardware-backed. |

**Argon2id can be added later** (via `@noble/hashes`, one audited dependency). Bitwarden itself allows switching KDF. For MVP, PBKDF2 at 600K is sufficient and OWASP-compliant.

---

## Architecture

### Key Hierarchy

```
Master Password (user-chosen)
    │
    └── PBKDF2(pw, email+salt, 600K iter) ──► Key Encryption Key (KEK) [256-bit]
                                                  │
                                                  │ wraps/unwraps
                                                  ▼
                             Data Encryption Key (DEK) [256-bit, randomly generated]
                                  │
                                  │ encrypts/decrypts
                                  ▼
                             Vault field data (account numbers, notes, documents, values)
```

### What's Stored on Anchora Servers

| Field | Format | Anchora can read? |
|-------|--------|-------------------|
| `salt` | Random bytes | Yes (needed for PBKDF2) |
| `wrapped_dek` | DEK encrypted with KEK (AES-256-GCM) | No — ciphertext without password |
| `recovery_wrapped_dek` | DEK encrypted with recovery-passphrase-derived key | No — ciphertext without passphrase |
| `tc_wrapped_dek` | DEK encrypted with TC's ECDH public key | No — ciphertext without TC's private key |
| Vault records | AES-256-GCM ciphertext | No — ciphertext without DEK |
| Asset categories, institution names | Plaintext | Yes (needed for guidance) |

---

## Flows

### 1. Vault Creation (Owner Signup)

```
1. Owner chooses Master Password
2. Client generates: salt = crypto.getRandomValues(new Uint8Array(32))
3. Client derives: KEK = PBKDF2(password, email+":"+salt, 600000, "SHA-256")
4. Client generates: DEK = crypto.getRandomValues(new Uint8Array(32))
5. Client wraps: wrapped_dek = AES-256-GCM(key=KEK, plaintext=DEK)
6. [Optional] Client generates recovery passphrase (10 words)
7. [Optional] recovery_key = PBKDF2(passphrase, salt2, 600000, "SHA-256")
8. [Optional] recovery_wrapped_dek = AES-256-GCM(key=recovery_key, plaintext=DEK)
9. Upload to server: salt, wrapped_dek, recovery_wrapped_dek (if any)
10. Server: stores auth hash + vault metadata. NEVER sees DEK, KEK, or password.
```

**Recovery passphrase shown ONCE, with instructions:**
> "Write these words down. Store them somewhere safe. Share them with your Trusted Contact in person. If they haven't accepted your invitation when you're no longer active, these words will give them access. Anchora has never seen these words and cannot recover them."

The passphrase is 10 random words from the BIP39 wordlist. The user is told to share it with their TC *in person* — this covers the "TC hasn't accepted yet" case without requiring server-side escrow.

### 2. Owner Login & Vault Access

```
1. Owner enters email + Master Password
2. Client downloads: salt, wrapped_dek
3. Client derives: KEK = PBKDF2(password, email+":"+salt, 600000, "SHA-256")
4. Client unwraps: DEK = AES-256-GCM-decrypt(key=KEK, ciphertext=wrapped_dek)
5. For reads: download encrypted vault records → decrypt with DEK
6. For writes: encrypt with DEK → upload ciphertext
7. DEK held in memory only. Never written to disk, localStorage, or sent to server.
```

### 3. Trusted Contact Designation (Bitwarden Flow)

```
1. Owner provides TC's email. Invitation sent.
2. TC receives email: "Gabby has designated you as a Trusted Contact on Anchora.
   Create your free account to complete the setup. Takes 2 minutes."

────────────────────────────────────────────────────────
   TC has NOT accepted yet:
   - tc_wrapped_dek does NOT exist
   - Vault is still encrypted and functional for Owner
   - Release cannot fire meaningfully
   - Owner sees "TC pending — ask them to accept" in the UI
────────────────────────────────────────────────────────

3. TC accepts → creates account:
   - Generates ECDH-P256 keypair in browser
   - Public key uploaded to server
   - Private key stays in browser, encrypted with TC's own KEK

4. Next time Owner logs in:
   - Client downloads TC's public key from server
   - Client unwraps DEK with Owner's KEK
   - Client wraps DEK with ECDH: ECDH(owner_ephemeral, TC_public) → shared secret
   - AES-256-GCM(key=shared_secret, plaintext=DEK) → tc_wrapped_dek
   - Upload tc_wrapped_dek + owner_ephemeral_public to server

5. Release fires:
   - TC downloads tc_wrapped_dek + owner_ephemeral_public
   - ECDH(TC_private, owner_ephemeral_public) → shared secret
   - DEK = AES-256-GCM-decrypt(key=shared_secret, ciphertext=tc_wrapped_dek)
   - TC decrypts vault records with DEK
```

### 4. Release When TC Hasn't Accepted (The Edge Case)

```
If release fires AND tc_wrapped_dek doesn't exist:

1. TC notified: "You were designated as Gabby's Trusted Contact but never accepted
   the invitation. If Gabby shared a recovery passphrase with you in person,
   enter it below to access their vault."

2. TC enters recovery passphrase → PBKDF2(passphrase, salt2, 600K) → recovery_key
   → AES-256-GCM-decrypt(recovery_wrapped_dek) → DEK
   → decrypt vault

3. If no recovery passphrase was set:
   "Your access cannot be granted because the account owner didn't complete
   setup. Contact Anchora support for guidance."
```

**This is the only clean way to handle this edge case.** No other industry solution solves "TC hasn't registered yet" without requiring TC registration. The recovery passphrase is the owner's mechanism for ensuring access regardless.

---

## Security Properties

| Property | Guarantee |
|----------|-----------|
| **Anchora cannot read vault** | ✅ DEK never leaves the client in plaintext. Server has only ciphertext + wrapped DEKs. |
| **Breach = useless data** | ✅ Database dump yields: encrypted vault fields + wrapped DEK. Without password, nothing is decryptable. |
| **Password compromised** | ⚠️ Attacker who knows password AND gets encrypted DB can decrypt. Mitigation: PBKDF2 600K iterations = ~1 second derivation per attempt = slow brute-force. |
| **TC loses private key** | TC can use recovery passphrase if owner shared it. |
| **Owner loses password** | Irrecoverable. Vault must be recreated. Clear warning during setup. |
| **TC hasn't accepted, release fires** | Recovery passphrase path. If none set, TC cannot access. |

---

## Implementation Plan

### Phase 1: Core Zero-Knowledge Vault (Sprint 1)

**Client (`@anchora/crypto` or inline):**

```typescript
// Zero dependencies. Web Crypto API only.

const PBKDF2_ITERATIONS = 600_000;
const AES_KEY_LENGTH = 256;
const SALT_LENGTH = 32;
const DEK_LENGTH = 32;

// Derive KEK from password
async function deriveKEK(password: string, email: string, salt: Uint8Array): Promise<CryptoKey> {
  const keyMaterial = await crypto.subtle.importKey(
    'raw',
    new TextEncoder().encode(password + ':' + email),
    'PBKDF2',
    false,
    ['deriveKey']
  );
  return crypto.subtle.deriveKey(
    { name: 'PBKDF2', salt, iterations: PBKDF2_ITERATIONS, hash: 'SHA-256' },
    keyMaterial,
    { name: 'AES-GCM', length: AES_KEY_LENGTH },
    false,
    ['wrapKey', 'unwrapKey', 'encrypt', 'decrypt']
  );
}

// Generate fresh DEK
async function generateDEK(): Promise<CryptoKey> {
  return crypto.subtle.generateKey(
    { name: 'AES-GCM', length: AES_KEY_LENGTH },
    true,  // extractable (needed for wrapKey)
    ['encrypt', 'decrypt', 'wrapKey']
  );
}

// Wrap DEK with KEK
async function wrapDEK(dek: CryptoKey, kek: CryptoKey): Promise<ArrayBuffer> {
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const wrapped = await crypto.subtle.wrapKey('raw', dek, kek, { name: 'AES-GCM', iv });
  // Prepend IV to wrapped data
  const result = new Uint8Array(iv.length + wrapped.byteLength);
  result.set(iv, 0);
  result.set(new Uint8Array(wrapped), iv.length);
  return result.buffer;
}

// Unwrap DEK with KEK
async function unwrapDEK(wrappedData: ArrayBuffer, kek: CryptoKey): Promise<CryptoKey> {
  const data = new Uint8Array(wrappedData);
  const iv = data.slice(0, 12);
  const wrapped = data.slice(12);
  return crypto.subtle.unwrapKey(
    'raw', wrapped, kek,
    { name: 'AES-GCM', iv },
    { name: 'AES-GCM', length: AES_KEY_LENGTH },
    false,  // non-extractable once unwrapped (safer for memory)
    ['encrypt', 'decrypt']
  );
}

// Encrypt vault field with DEK
async function encryptField(plaintext: string, dek: CryptoKey): Promise<{ iv: Uint8Array; ciphertext: ArrayBuffer }> {
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const ciphertext = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    dek,
    new TextEncoder().encode(plaintext)
  );
  return { iv, ciphertext };
}

// Decrypt vault field with DEK
async function decryptField(iv: Uint8Array, ciphertext: ArrayBuffer, dek: CryptoKey): Promise<string> {
  const plaintext = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv },
    dek,
    ciphertext
  );
  return new TextDecoder().decode(plaintext);
}

// Generate recovery passphrase (10 words, BIP39)
function generateRecoveryPassphrase(): string {
  const words = BIP39_WORDLIST; // embedded; ~25KB
  const indices = crypto.getRandomValues(new Uint16Array(10));
  return Array.from(indices, i => words[i % words.length]).join(' ');
}

// Generate ECDH keypair for TC
async function generateTCKeypair(): Promise<CryptoKeyPair> {
  return crypto.subtle.generateKey(
    { name: 'ECDH', namedCurve: 'P-256' },
    true,  // extractable (needed to wrap private key)
    ['deriveKey']
  );
}

// Wrap DEK with TC's public key (ECDH + AES-GCM)
async function wrapDEKForTC(dek: CryptoKey, tcPublicKey: CryptoKey): Promise<{
  ownerEphemeralPublic: ArrayBuffer;
  tcWrappedDEK: ArrayBuffer;
}> {
  // Generate ephemeral keypair
  const ephemeral = await crypto.subtle.generateKey(
    { name: 'ECDH', namedCurve: 'P-256' },
    true,
    ['deriveKey']
  );
  // Derive shared secret
  const sharedSecret = await crypto.subtle.deriveKey(
    { name: 'ECDH', public: tcPublicKey },
    ephemeral.privateKey,
    { name: 'AES-GCM', length: 256 },
    false,
    ['wrapKey']
  );
  // Wrap DEK
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const wrapped = await crypto.subtle.wrapKey('raw', dek, sharedSecret, { name: 'AES-GCM', iv });
  // Export ephemeral public key
  const ephemeralPublicRaw = await crypto.subtle.exportKey('raw', ephemeral.publicKey);
  // Prepend IV
  const result = new Uint8Array(iv.length + wrapped.byteLength);
  result.set(iv, 0);
  result.set(new Uint8Array(wrapped), iv.length);
  return {
    ownerEphemeralPublic: ephemeralPublicRaw,
    tcWrappedDEK: result.buffer
  };
}
```

**Backend changes:**

1. New DB columns: `salt`, `wrapped_dek`, `recovery_wrapped_dek`, `recovery_salt`
2. Vault records table: `encrypted_data` (was `plaintext` before) + `iv` column per field
3. Remove server-side AES-256 encryption (now client-side)
4. KMS integration removed for field encryption
5. `POST /v1/vault-records` → accepts pre-encrypted fields

### Phase 2: TC Key Handoff (Sprint 2)

1. Invitation flow: TC creates account → generates ECDH keypair → uploads public key
2. Owner re-login → downloads TC public key → wraps DEK for TC → uploads `tc_wrapped_dek`
3. Release flow: TC downloads `tc_wrapped_dek` → ECDH decrypt → DEK → access vault

### Phase 3: Recovery Passphrase (Sprint 2, alongside TC)

1. Optional during vault creation
2. 10-word BIP39 passphrase, shown once
3. `recovery_wrapped_dek` stored on server
4. TC can use recovery passphrase instead of ECDH if TC hasn't accepted

---

## File & Document Encryption

Supporting documents (PDFs, images) uploaded to R2 get the same treatment:

```
1. File selected in browser
2. Client generates file-specific key (FK = random 32 bytes)
3. File encrypted with AES-256-GCM(key=FK)
4. Encrypted file uploaded to R2
5. FK encrypted with DEK → stored in vault record alongside R2 path
6. On access: decrypt FK with DEK → decrypt file in browser
```

For large files, use streaming: `crypto.subtle.encrypt` on chunks (AES-GCM streaming via `{ name: 'AES-CTR' }` + HMAC, or chunked AES-GCM).

---

## Nigerian Mobile Browser Compatibility

| Browser | Web Crypto | PBKDF2 | AES-GCM | ECDH | Market |
|---------|-----------|--------|---------|------|--------|
| Chrome Android | ✅ | ✅ | ✅ | ✅ | ~60% |
| Safari iOS | ✅ | ✅ | ✅ | ✅ | ~20% |
| Samsung Internet | ✅ | ✅ | ✅ | ✅ | ~8% |
| Opera Mini | ❌ | ❌ | ❌ | ❌ | ~8% |
| UC Browser | ⚠️ | ⚠️ | ⚠️ | ⚠️ | ~3% |

**Decision:** Require Chrome, Safari, or Samsung Internet. Display warning on Opera Mini/UC Browser: "Your browser doesn't support the encryption required for zero-knowledge vault security. Please use Chrome, Safari, or Samsung Internet."

This matches the target audience (Nigerian professionals, founders, investors) who overwhelmingly use modern browsers.

---

## Comparison: This vs. Competitors

| Feature | Anchora (This Spec) | Bitwarden | 1Password | Everplans |
|---------|--------------------|-----------|-----------|-----------|
| Zero-knowledge | ✅ DEK/KEK envelope | ✅ Same model | ✅ Secret Key + MPW | ✅ Client-side |
| Trusted Contact handoff | ✅ Bitwarden EA model | ✅ RSA-OAEP | ✅ ECDH vault share | Via account |
| TC hasn't accepted yet | ✅ Recovery passphrase | ❌ Can't release | ❌ Can't release | ❌ Can't release |
| Dependencies | ✅ Zero (Web Crypto only) | | WASM (Argon2) | |
| Key derivation | PBKDF2 600K | PBKDF2 600K default | PBKDF2 + Secret Key | |
| TC asym | ECDH-P256 + AES-GCM | RSA-OAEP | X25519 | N/A |

---

## Why Not...

| Approach | Why rejected |
|----------|-------------|
| Shamir's Secret Sharing | Complex. Requires share delivery. TC must store shares. No industry products use this for UX. |
| Passkeys (WebAuthn PRF) | Not universally supported (no Opera Mini, UC Browser). PRF extension is Chrome-only. Can add later. |
| libsodium.js | 200KB WASM. Web Crypto already provides PBKDF2+AES-GCM+ECDH natively. Adds no benefit. |
| Argon2id for MVP | Adds a dependency. PBKDF2 600K is Bitwarden's default and OWASP-recommended. Argon2id upgrade path later. |
| Server-side key escrow | Breaks zero-knowledge. Anchora must not be able to decrypt. |
| TC auto-registration (server creates TC keypair) | Server holds TC private key at creation → can decrypt. Breaks zero-knowledge. |

---

## Summary

| Question | Answer |
|----------|--------|
| What crypto library? | **None.** Web Crypto API. Zero dependencies. |
| How does the owner decrypt? | Master Password → PBKDF2 600K → KEK → unwrap DEK |
| How does the TC decrypt? | ECDH private key (created on acceptance) OR recovery passphrase |
| What if TC hasn't accepted? | Recovery passphrase path. If none set, TC cannot access (correct behavior). |
| Is this industry standard? | Yes. Bitwarden Emergency Access model. |
| Is it simpler than the previous spec? | Yes. No Shamir. No passkeys. No WASM deps. No key splitting. |
| Can it stand the test of time? | Yes. PBKDF2 + AES-GCM + ECDH are all Web Platform standards. |