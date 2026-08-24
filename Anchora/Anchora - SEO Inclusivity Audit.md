# Anchora — SEO Inclusivity Audit

**Date:** 2026-08-24  
**URL:** https://anchora.com.ng  
**Stack:** Next.js (SSG/SSR inferred), deployed on anchora.com.ng with app.anchora.com.ng subdomain

---

## Executive Summary

The landing page has solid content fundamentals but misses critical technical SEO infrastructure and entirely lacks inclusivity signals. The site is invisible to search engines for key discovery paths (no sitemap, no robots.txt, no structured data) and inaccessible to Nigeria's non-English-speaking majority (~60% of the population). Fixing the technical gaps is a ~2-hour engineering task with outsized ROI.

---

## 🔴 Critical (Blocking Search Discovery)

### 1. No `robots.txt` — 404
`https://anchora.com.ng/robots.txt` returns a 404. Search engines can't read crawling directives. Without it, crawlers may waste budget on app routes or miss the landing page entirely.

**Fix:** Create `/public/robots.txt` or serve via Next.js route:
```
User-agent: *
Allow: /
Sitemap: https://www.anchora.com.ng/sitemap.xml
```

### 2. No `sitemap.xml` — 404
`https://anchora.com.ng/sitemap.xml` returns a 404. This is the single biggest discoverability blocker — Google, Bing, and all crawlers use sitemaps as the entry point for indexing.

**Fix:** Generate with `next-sitemap` or Next.js built-in `sitemap.ts`:
```ts
// app/sitemap.ts
export default function sitemap() {
  return [
    { url: 'https://www.anchora.com.ng', lastModified: new Date(), changeFrequency: 'weekly', priority: 1 },
    { url: 'https://www.anchora.com.ng/privacy', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.3 },
    { url: 'https://www.anchora.com.ng/terms', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.3 },
    { url: 'https://www.anchora.com.ng/security', lastModified: new Date(), changeFrequency: 'monthly', priority: 0.5 },
  ];
}
```

### 3. No canonical URL
No `<link rel="canonical">` on the page. This leaves the site vulnerable to duplicate content penalties if the page is accessible at both `anchora.com.ng` and `www.anchora.com.ng`.

**Fix:** Add to layout `head.tsx` or `metadata` export:
```tsx
alternates: { canonical: 'https://www.anchora.com.ng' }
```

### 4. No structured data (JSON-LD)
Zero schema.org markup — no `Organization`, `WebSite`, `SoftwareApplication`, or `FAQ` structured data. This means:
- No rich results in Google (sitelinks, logo, description)
- No knowledge panel eligibility
- Competitors with schema will outrank for branded queries

**Fix:** Add at minimum:
```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Anchora",
  "url": "https://www.anchora.com.ng",
  "description": "Your digital financial legacy vault — document your assets and designate a trusted contact for guided discovery.",
  "inLanguage": "en-NG"
}
```
Plus `Organization` schema with logo, sameAs (social profiles), and `SoftwareApplication` for the vault product.

---

## 🟠 High (Social & Visual Discovery)

### 5. Missing Open Graph / Twitter images
- `og:image` — **missing**
- `twitter:image` — **missing**
- `og:url` — **missing**

Social shares show a plain text card with no preview image. In Nigeria, WhatsApp and Twitter/X shares are a primary distribution channel.

**Fix:** Create a 1200×630px OG image (Anchora logo + tagline on brand background), host at `/images/og-default.png`, and add to metadata:
```tsx
openGraph: {
  images: [{ url: '/images/og-default.png', width: 1200, height: 630 }],
  url: 'https://www.anchora.com.ng',
},
twitter: {
  card: 'summary_large_image',
  images: ['/images/og-default.png'],
}
```

### 6. Twitter card is `summary` not `summary_large_image`
The `summary` card shows a tiny thumbnail. `summary_large_image` is the standard for product landing pages and shows a full-width image.

### 7. Empty first meta tag
There's an empty `<meta>` tag (Next.js artifact) that adds noise and may confuse crawlers.

---

## 🟡 Medium (Content & Reach)

### 8. English-only — no Nigerian language support
Nigeria has 500+ languages. ~60% of the population doesn't use English as a primary language. The site serves zero content in:
- **Nigerian Pidgin English** (~75M speakers — the actual lingua franca)
- **Hausa** (~70M, dominant in the North)
- **Yoruba** (~45M, dominant in the Southwest)
- **Igbo** (~40M, dominant in the Southeast)

Even a single Pidgin landing page would 10× the addressable audience.

**Fix (phased):**
1. **Phase 1:** Add `hreflang` tags for en-NG and en (current page)
2. **Phase 2:** Translate landing page to Nigerian Pidgin; add `hreflang="pcm-NG"`
3. **Phase 3:** Yoruba, Hausa, Igbo key pages (pricing, how it works)

### 9. No blog or content section
Zero content marketing surface. Blog content is the primary vehicle for:
- Long-tail SEO keywords ("how to document assets in Nigeria", "Nigerian pension discovery process")
- Problem education (your GTM pillar #2 at 25%)
- Backlink acquisition

**Fix:** Add `/blog` with a Next.js MDX setup. Start with 3-5 foundational posts targeting Nigerian financial literacy keywords.

### 10. Missing keyword targeting for Nigerian search intent
Current meta keywords: `financial legacy planning, digital vault, trusted contact management, legacy planning Nigeria`

Missing high-volume Nigerian search terms:
- "how to write a will in Nigeria"
- "next of kin bank account Nigeria"
- "Nigerian pension RSA"
- "dormant account Nigeria"
- "estate planning Nigeria"
- "digital asset inheritance"
- "what happens to my bank account when I die Nigeria"

### 11. H1 is a tagline, not a keyword-rich heading
`"Your wealth deserves a guardian that never sleeps"` — evocative but zero SEO value. The H1 should include primary keywords.

**Consider:** `"Anchora — Nigeria's Digital Legacy Vault for Financial Asset Discovery"` or similar keyword-rich variant.

---

## 🟢 Low (Technical Polish)

### 12. Font preloads missing `crossorigin`
All 6 font preloads lack `crossorigin="anonymous"`. Browsers will re-download fonts, wasting bandwidth and slowing LCP on mobile (critical in Nigeria where 3G/4G is common).

### 13. No skip-to-content link
Accessibility best practice and also a ranking signal. Add a visually hidden skip link.

### 14. Only 2 images on the page
For a product page, this is sparse. More images with descriptive alt text would improve:
- Image search discovery
- Dwell time
- Accessibility for screen reader users

### 15. Missing privacy/terms/security pages in footer blog links
These exist but aren't crawled efficiently without sitemap entries.

---

## Inclusivity-Specific Recommendations

| Dimension | Current State | Target State |
|-----------|--------------|--------------|
| **Language** | English only | Pidgin + en-NG hreflang; phase in Yoruba/Hausa/Igbo |
| **Literacy level** | University-level English | Plain language (Flesch-Kincaid grade 8-9); Pidgin variant |
| **Mobile experience** | OK (responsive) | Optimize for 3G; <3s LCP on slow connections |
| **Cultural framing** | Western "estate planning" | Nigerian: family, community, "not leaving wahala for your people" |
| **Religious sensitivity** | Neutral | Consider Islamic inheritance context (Northern Nigeria) |
| **Accessibility** | Basic | Add WCAG 2.1 AA: skip link, focus indicators, alt text, ARIA labels |
| **Trust signals** | Security section exists | Add NDPR compliance badge, Nigerian hosting mention, local testimonials |
| **Payment visibility** | Naira pricing ✓ | Add mobile money / USSD payment mention (critical for inclusion) |

---

## Priority Action Plan

### 🔴 This week (blocking):
1. Add `robots.txt` — 15 min
2. Add `sitemap.xml` — 30 min (with `next-sitemap`)
3. Add canonical URL — 5 min
4. Add JSON-LD structured data (Organization + WebSite) — 30 min

### 🟠 Next week (high impact):
5. Create OG image and wire into metadata — 1 hr
6. Switch Twitter card to `summary_large_image` — 5 min
7. Add Nigerian Pidgin landing page variant + hreflang — 3-4 hrs
8. Add FAQ section with FAQPage schema — 2 hrs

### 🟡 This month (content flywheel):
9. Launch `/blog` with 5 SEO-targeted articles — 2 days
10. Translate key pages to Pidgin — 1 week
11. Add Nigerian keyword research and optimize headings — 1 day

### 🟢 Next quarter:
12. Hausa/Yoruba/Igbo key pages
13. Video content with transcripts
14. Accessibility audit and WCAG compliance

---

## Competitor Comparison

Sites to benchmark against for Nigerian fintech SEO:
- **piggyvest.com** — Strong blog content strategy, Nigerian keywords
- **cowrywise.com** — Good structured data, content marketing
- **flutterwave.com** — Multi-language, strong technical SEO
- **paystack.com** — Excellent content + developer docs SEO

None of these are in the legacy/digital vault space, giving Anchora a first-mover advantage in SEO if the gaps are closed quickly.

---

## Appendix: Current Meta Snapshot

```
Title:        "Anchora - Your Digital Financial Legacy Vault" (60 chars — good length)
Description:  "Document your financial and physical assets and designate a trusted 
               contact who is guided through discovery via a staged, inactivity-triggered 
               release workflow." (161 chars — over 160, truncated in SERPs)
Keywords:     "financial legacy planning, digital vault, trusted contact management, 
               legacy planning Nigeria"
OG Title:     Same as title ✓
OG Desc:      "Your wealth deserves a guardian that never sleeps." 
              (Different from meta description — inconsistent)
Twitter:      Same as OG (inherits the short desc)
Canonical:    MISSING
Robots:       MISSING
Sitemap:      MISSING
Schema:       MISSING
Lang:         "en" (should be "en-NG")
```