# Anchora — Homepage Consolidation Spec

**Date:** 2026-09-04
**Context:** SEO teardown flagged 3 sections ("How it works", "The experience", "Inactivity engine") as repeating the same mechanism. Redundancy check found additional overlaps: asset categories listed 3×, "never moves money" stated 4×, encryption claim in 2 places.
**Status:** Spec — ready for implementation

---

## Current vs Proposed

| Current Homepage (9 sections) | Proposed (8 sections, deduplicated) |
|------|------|
| 1. Hero | 1. Hero (rewritten — see Access-Control Spec) |
| 2. The problem (3 long cards) | 2. Recognition strip (8 icons — replaces old categories section) |
| 3. How it works (3 steps) | 3. The problem (1 concise paragraph) |
| 4. Asset categories | 4. How it works (3 steps, merged from 3 old sections) |
| 5. The experience (3 steps again) 🔥 | 5. Security + legal boundary (rewritten) |
| 6. Inactivity engine (5 stages again) 🔥 | 6. Pricing |
| 7. Security | 7. FAQ (collapsed, 8 questions) |
| 8. Pricing | 8. Final CTA |
| 9. Final CTA | |

**Killed:** "The experience" + "Inactivity engine" (duplicates of how-it-works) + "Asset categories" (duplicate of recognition strip)

**Result:** 9 sections → 8. Two whole sections removed, one new one added, net -1 section, ~40 fewer lines.

---

## Redundancies Found & Fixed

| Repeated phrase | Appears in | Fix |
|----------------|-----------|-----|
| Bank/pension/property/investment list | Hero, recognition strip, problem, categories section | Merge recognition strip + categories. Rewrite problem without list. |
| "Never holds/moves your money" | Hero trust line, step 1, categories, security | Keep hero trust line (1 line) + security section (where it belongs). Remove from step 1 and categories. |
| "Every field encrypted" | Step 1, security section | Remove from step 1. Security section owns encryption. |

---

## Section-by-Section Rewrite

### Section 1: Hero

From Access-Control Spec. Outcome-led.

```
## Give your family a clear map to every asset you own.

Record your bank accounts, pensions, investments, property, crypto, and
important documents in one secure place. Designate a trusted person who
is guided step-by-step through finding every record when the time comes.

[Create your free asset inventory]  [See how it works ▾]

Free for 3 assets · No card required
```

The asset list in the subheadline is intentional — it's the only place the visitor's eyes go first. Recognition strip immediately below reinforces it visually.

---

### Section 2: Recognition Strip (Replaces Old "Asset Categories" Section)

**Purpose:** In 3 seconds, the visitor matches their life to the product. No scroll required.

```
[🏦 Bank Accounts]  [📊 Investments]  [🏠 Property]      [💼 Pensions]
[🪙 Crypto]         [📄 Documents]     [🚗 Vehicles]      [🌍 Foreign]
                                         ↓ see all 11 categories
```

**Implementation:** 
- 8 icons in a 4×2 grid. Icon + label only. No descriptions.
- "See all 11 categories" link scrolls to a lightbox or dedicated `/categories` page (not a new homepage section).
- Icons from Lucide or Heroicons. No custom imagery.
- This IS the asset categories. No separate section needed.

**Why this works:** The visitor mentally checks boxes silently — "Yes, bank. Yes, crypto. Yes, property." — without reading a word of copy. The old categories section (11 cards with descriptions) required scrolling and reading. This doesn't.

---

### Section 3: The Problem (Consolidated)

**Before:** Three large cards, each with a statistic, subheading, and body. ~20 lines.

**After:** One paragraph. No asset-type list (the recognition strip just did that). No statistics (move to blog).

```
## The problem

Your family cannot recover an asset they do not know exists.

Most people's financial lives are scattered — accounts split across
banks, investments on different platforms, property papers in a drawer,
pension records never discussed. If something happens to you, the
person left to figure it out has to guess what you had, where it is
held, and what each institution requires.

Anchora turns that scattered picture into one organised, secure
record — before your family has to search for it under pressure.
```

**Why no asset list here:** The recognition strip 3 inches above just showed bank accounts, investments, property, pensions, crypto, documents, vehicles, foreign accounts. Listing them again in the problem paragraph ("A bank account in one place. A pension somewhere else...") is the same repetition we're removing.

**Statistics moved to:** Blog post "What happens to dormant accounts in Nigeria" + pensions use-case page. Keep the punch of the stats, just not on the homepage where they require footnotes.

---

### Section 4: How It Works (Merged — The One True Version)

Replaces three old sections: "How it works," "The experience," "Inactivity engine."

```
## How it works

### 1. Record what exists
Add the institution, supporting documents, and the instructions your
family will need. You decide what goes in and who eventually sees it.

Takes as little as 10 minutes. Free for your first 3 assets.

### 2. Choose your trusted contact
Designate the one person you trust to be guided through discovery.
Set how long Anchora waits before reaching out — 3, 6, 12, or
18 months. They're notified but cannot see anything until it's time.

### 3. They're guided when the time comes
Stay active, and your information stays private. If Anchora detects
no activity for the period you set, a careful, staged process begins.
You are warned. You are given time to respond. Your trusted contact
must verify their identity. Only then does guided discovery begin.

Any activity from you at any stage cancels everything immediately.
```

**What's deliberately NOT here (found in other sections):**

| Claim | Lives in |
|-------|----------|
| "Anchora never holds or moves your money" | Security section (Section 5) |
| "Every field is encrypted" | Security section (Section 5) |
| "AES-256-GCM at rest, TLS 1.3 in transit" | Security section (Section 5) |
| 5-stage inactivity timeline | No longer on homepage. Full detail on `/how-it-works` page. |
| "No legal battles, no guessing" | Removed entirely (replaced with honest language) |

**What's removed from old homepage:**
- ~~"The experience" section~~ (duplicate of these 3 steps)
- ~~"Inactivity engine" section~~ (collapsed into step 3's paragraph + `/how-it-works` page)
- ~~"Anchora does not connect to, control, or move your funds"~~ (lives in security now)
- ~~"Every sensitive field you enter is encrypted"~~ (lives in security now)
- ~~"No separate check-in action required"~~ (folded into "Stay active, and your information stays private")

---

### Section 5: Security + Legal Boundary

From Access-Control Spec. This is where encryption, access controls, and money-disclaimer live — ONCE.

```
## Your data. Your control.

No human at Anchora can read your vault. Our encryption keys are
restricted by infrastructure policy to the automated release
pipeline only. AES-256-GCM at rest. TLS 1.3 in transit. Complete
immutable audit trail.

### What Anchora is — and is not

Anchora records and organises information about your assets.
It is not a bank, custodian, executor, law firm, will, or
grant of probate. Anchora never holds or moves your money.

A trusted contact may still need a will, probate, letters of
administration, or other legal documents required by each
institution. Anchora provides the map — not the keys to the vault.

[Learn more about our security →]   [Read the FAQ →]
```

---

### Section 6: Pricing

Keep existing. Two fixes:
- Clarify what "standard release" means (which verification steps are included in Free)
- Show first-year annual effective cost, not just lump sum

---

### Section 7: FAQ (Collapsed)

From Access-Control Spec. 8 collapsed questions. Full FAQ at `/faq`.

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

### Section 8: Final CTA

```
## Give your family a map, not a mystery.

Create your free vault in minutes. Documenting what you have takes
less time than your family will spend trying to figure it out without you.

[Create your free asset inventory]  ·  Free for 3 assets  ·  No card required
```

---

## Redundancy Audit (Each Claim Appears Once)

| Claim / Concept | Where it lives | Not repeated in |
|----------------|---------------|-----------------|
| Asset categories list | Hero (text) + Recognition strip (icons) | Problem, How it works |
| "Never holds/moves money" | Security section | Hero, Step 1, Categories |
| Encryption details (AES, TLS) | Security section | Step 1, FAQ |
| "No human can read vault" | Security section | How it works (just says "your information stays private") |
| Release mechanism detail | `/how-it-works` page (linked from step 3) | FAQ (yes — "Can inactivity release?" — but FAQ is a separate utility, not narrative) |
| Product boundary (not a will, not a bank) | Security section | Hero, How it works |
| Free tier (3 assets) | Hero CTA + How it works step 1 | (Repeated deliberately — CTA needs it, step 1 qualifies scope) |
| Inactivity window (3-18 months) | How it works step 2 | FAQ (acceptable — separate utility) |

**The rule:** Every concept gets ONE primary home in the narrative. The only intentional repeats are: free tier (CTA context + feature context) and FAQ overlap (separate utility format, not narrative flow).

---

## Section Inventory: Before vs After

| Before | Lines (approx) | After | Lines (approx) |
|--------|-------|--------|-------|
| Hero (vague) | ~8 | Hero (outcome-led) | ~6 |
| Problem (3 cards) | ~20 | Recognition strip | ~4 (NEW, replaces categories) |
| How it works | ~15 | Problem (1 paragraph) | ~5 |
| Asset categories | ~20 | How it works (merged) | ~12 |
| The experience 🔥 | ~12 | Security + boundary | ~12 |
| Inactivity engine 🔥 | ~18 | Pricing | ~16 |
| Security (vague) | ~24 | FAQ (collapsed) | ~10 |
| Pricing | ~16 | Final CTA | ~4 |
| Final CTA | ~4 | — | — |
| **Total** | **~137** | **Total** | **~69** |

**Result:** ~50% shorter. Zero repeated concepts. Every claim has one home.

---

## Implementation Notes

1. The old "Asset categories" section content (11 categories with descriptions) moves to a dedicated `/categories` page. Link to it from the recognition strip's "See all 11 categories →."

2. The old "Inactivity engine" 5-stage content moves to the `/how-it-works` page. Keep it — it's useful detail, just not on the homepage.

3. The recognition strip is a thin `<section>` with 8 `<div>` cards in a CSS grid. Each card: icon (SVG, 24×24) + label text. No images. No hover effects needed.

4. FAQ uses `<details>` elements. No JavaScript. Progressive enhancement — works in every browser.

5. All copy follows Anchora conventions: no em dashes, "Trusted Contact" label, "discovery" not recovery.

6. Statistics (dormant accounts, PenCom) live in blog posts — not the homepage. Publish at least one before removing from the current site.