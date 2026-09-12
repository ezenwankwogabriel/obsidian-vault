# Anchora — Homepage Consolidation Spec

**Date:** 2026-09-04
**Context:** SEO teardown flagged 3 sections ("How it works", "The experience", "Inactivity engine") as repeating the same mechanism. Three sections all say the same thing: record assets → designate contact → guided discovery after inactivity. Consolidate into one.
**Status:** Spec — ready for implementation

---

## Current vs Proposed

| Current Homepage (9 sections) | Proposed (10 sections, consolidated) |
|------|------|
| 1. Hero | 1. Hero (rewritten — see Access-Control Spec) |
| 2. The problem (3 long cards) | 2. **Recognition strip** (NEW — logos/icons) |
| 3. How it works (3 steps) | 3. **The problem** (1 concise paragraph) |
| 4. Asset categories | 4. **How it works** (3 steps, merged from 3 old sections) |
| 5. The experience (3 steps again) 🔥 | 5. Asset categories |
| 6. Inactivity engine (5 stages again) 🔥 | 6. **Security + legal boundary** (rewritten) |
| 7. Security | 7. Pricing |
| 8. Pricing | 8. FAQ (collapsed, 8-10 questions) |
| 9. Final CTA | 9. Final CTA |
| — | 10. Footer links (security, privacy, terms, contact) |

**Killed:** "The experience" section + "Inactivity engine" full section

**Merged:** How it works now carries the full story once, with inactivity stages as a small collapsed detail

**New:** Recognition strip after hero

---

## Section-by-Section Rewrite

### Section 1: Hero

Already spec'd in [Anchora - Trust Positioning & Access Control Spec](./Anchora%20-%20Trust%20Positioning%20&%20Access%20Control%20Spec.md), section 1.1. Outcome-led, no vague "Make sure your loved ones know what you own."

```
Give your family a clear map to every asset you own.

Record your bank accounts, pensions, investments, property, crypto, and
important documents in one secure place. Designate a trusted person who
is guided step-by-step through finding every record when the time comes.
```

---

### Section 2: Recognition Strip (NEW)

**Purpose:** In 3 seconds, the visitor sees themselves in the product. No scrolling required.

**Format:** Row of icons with labels — no descriptions. Pure pattern matching.

```
[🏦 Bank Accounts]  [📊 Investments]  [🏠 Property]  [💼 Pensions]
[🪙 Crypto]         [📄 Documents]     [🚗 Vehicles]  [🌍 Foreign Accounts]
    [+ 4 more categories →]
```

This is a rapid-fire visual checklist. The visitor mentally checks boxes: "Yes, I have that. Yes, and that. Oh, they cover crypto too." It creates recognition before we explain anything.

**Implementation:** 8 category icons in a row. "See all 12 categories →" link. No descriptions. Icons only. This takes zero cognitive load.

---

### Section 3: The Problem (Consolidated)

**Before:** Three large cards with heavy statistics, redundant framing, and sourced-but-unlinked claims.

**After:** One concise paragraph. Keep the emotional hook. Drop the statistics to the blog (where they can be properly sourced and dated).

```
## The problem

Your family cannot recover an asset they do not know exists.

A bank account in one place. A pension somewhere else. Property papers in
a drawer. An investment app only you know about. If something happens to
you, the person left to figure it out has to guess what you had, where
it is held, and what they need to do.

Anchora turns that scattered financial life into one organised, secure
record — before your family has to search for it under pressure.
```

**What's removed:**
- ~~"19.7 million dormant accounts"~~ → moves to blog post with proper source + date
- ~~"92% of Personal Pension Plan accounts unfunded"~~ → moves to pensions use-case page
- Three cards → one paragraph. Less to read. Same emotional weight.
- "Would anyone even know what you have?" → folded into the opening sentence
- "Is there one person actually responsible?" → implied by "the person left to figure it out"
- "Would they know how to actually get it?" → implied by "have to guess what they need to do"

---

### Section 4: How It Works (Merged — The One True Version)

This replaces three separate sections: "How it works", "The experience", and "Inactivity engine."

```
## How it works

Three steps. One secure record. No confusion when it matters.

### 1. Record what exists
Add the institution, asset, supporting documents, and the instructions your
family will need. Anchora does not connect to, control, or move your funds.
Every sensitive field you enter is encrypted. You control what goes in.

Takes as little as 10 minutes to set up. Free for your first 3 assets.

### 2. Choose your trusted contact
Designate the one person you trust to be guided through discovery. Set how
long Anchora waits before reaching out — 3, 6, 12, or 18 months. Your
trusted contact is notified but cannot see anything until it is time.

You can change your trusted contact at any time.

### 3. They're guided when the time comes
Stay active, and your information stays private — no separate check-in
required. If Anchora detects no activity for the period you set, a
careful, staged verification process begins before anything is released.

[See exactly how release works ▸]

(Clicking expands inline:)
  • We warn you — multiple emails over several days
  • We wait — any activity from you cancels everything immediately
  • We verify — your trusted contact must confirm their identity
  • We guide — step by step through each asset, not a raw data dump
  • You can cancel at any stage by logging in
```

**What's removed:**
- ~~"The experience" section~~ — this IS the experience
- ~~5-stage inactivity timeline~~ → collapsed into "See exactly how release works"
- ~~"No legal battles, no guessing"~~ → replaced with honest language
- ~~Step 3 "Stay active, stay in control" as a standalone point~~ → folded into step 3's middle paragraph
- ~~Repetition of "you control what goes in" in step 1~~ → said once

**What's kept:**
- Three steps (record → designate → guide)
- The free tier mention in step 1
- The inactivity window options
- The "never holds your money" disclaimer
- The staged release details (collapsed, not scrolling)

---

### Section 5: Asset Categories

**Keep as-is.** This section works. It helps visitors see themselves in the product. Minor fixes only:

- Remove "Jewelry & Watches" if not ready (avoid overpromise)
- Remove "Other" — it dilutes the specificity
- Add "Documents" as a standalone category (title deeds, certificates, wills)

```
## Every account. One place.

[Bank Account] [Investment Platform] [Crypto Wallet] [Pension Portal]
[Insurance Policy] [Foreign Account] [Real Estate] [Vehicle]
[Documents] [Share Certificates] [Subscription]

11 categories. You record what you have. Anchora never moves your money.
```

---

### Section 6: Security + Legal Boundary

Already spec'd in the Access-Control Spec. Replace current security section entirely.

```  
## Your data. Your control. No exceptions.

No human at Anchora can read your vault. Our encryption keys
are restricted by infrastructure policy to the automated
release pipeline only — no employee, founder, or administrator
has access. AES-256-GCM at rest. TLS 1.3 in transit. HSTS
enforced. Complete audit trail.

### What Anchora is — and is not

Anchora records and organises information about your assets.
It is not a bank, custodian, executor, law firm, will, or
grant of probate. A trusted contact may still need a will,
probate, letters of administration, or other legal documents
required by each institution.

Anchora never holds or moves your money.

[Learn more about our security →]
```

---

### Section 7: Pricing

Keep existing. Fix: clarify what "standard release" means. Show first-year annual equivalent. Note if activation fee is refundable.

---

### Section 8: FAQ (Collapsed)

Already spec'd in the Access-Control Spec. Show 8-10 most important questions collapsed. Full FAQ at `/faq`.

```
## Questions you're probably asking

▸ Is Anchora a will or a replacement for a lawyer?
▸ Can Anchora employees see what's in my vault?
▸ Does Anchora store passwords, PINs, or crypto seed phrases?
▸ Can inactivity alone release my data?
▸ What must my trusted contact prove?
▸ What happens to my records if Anchora closes?
▸ Can I export or permanently delete everything?
▸ Does Anchora work for assets outside Nigeria?

[See all questions →](/faq)
```

---

### Section 9: Final CTA

Keep existing. Minor copy tweak:

```
## Give your family a map, not a mystery.

Create your free vault in minutes. Documenting what you have takes less
time than your family will spend trying to figure it out without you.

[Create your free asset inventory]  •  Free for 3 assets  •  No card required
```

---

### Section 10: Footer Links

Ensure visible links to: Security, Privacy, Terms, FAQ, Contact, Blog.

---

## Section Inventory: Before vs After

| Before | Lines (approx) | After | Lines (approx) |
|--------|-------|--------|-------|
| Hero (vague) | ~8 | Hero (outcome-led) | ~6 |
| — | — | Recognition strip | ~4 (NEW) |
| Problem (3 cards) | ~20 | Problem (1 paragraph) | ~6 |
| How it works | ~15 | How it works (merged) | ~20 |
| Asset categories | ~20 | Asset categories (11) | ~18 |
| The experience 🔥 | ~12 | — | 0 (merged) |
| Inactivity engine 🔥 | ~18 | — | 0 (collapsed inside step 3) |
| Security (vague) | ~24 | Security + boundary | ~16 |
| Pricing | ~16 | Pricing | ~16 |
| Final CTA | ~4 | FAQ (collapsed) | ~10 (NEW) |
| — | — | Final CTA | ~4 |
| **Total** | **~137** | **Total** | **~100** |

**Result:** ~37 fewer lines. Zero repetitive sections. Product category communicated in the first 3 seconds. Trust questions answered before they become objections.

---

## Implementation Notes

1. The "Interactive TC preview" from the teardown's recommended order is deferred. It's high-effort (requires a live demo or animated mockup). The FAQ and security section address trust without it.

2. The recognition strip can start as simple icon + text (no custom imagery). SVG icons from Lucide or Heroicons. No custom illustration needed.

3. The collapsed "See exactly how release works" inline expand is a `<details>` element. No JavaScript required.

4. All copy follows Anchora conventions: no em dashes, "Trusted Contact" not executor/beneficiary, "discovery" not recovery.

5. The statistics (dormant accounts, PenCom) are removed from the homepage but should be published as a properly sourced blog post or use-case page before they're removed. Don't lose the content — move it where it can be properly cited.