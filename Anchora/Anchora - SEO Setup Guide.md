# Anchora — SEO Setup Guide (Next.js 14 App Router)

**Date:** 2026-08-24  
**Stack:** Next.js 14 (App Router), TypeScript, Tailwind, shadcn/ui  
**Pre-audit:** [[Anchora - SEO Inclusivity Audit]]

---

## Overview

This guide covers every SEO action for `anchora.com.ng`, ordered by impact. Phase 1 is blocking (search engines can't discover the site). Phase 2 is high-impact social/inclusivity. Phase 3 is the content flywheel.

All code is copy-paste ready. File paths assume the standard Next.js 14 App Router layout.

### 📋 Copy decision log

| Surface | Final copy | Rationale |
|---------|-----------|-----------|
| **On-image text** (Photopea) | "Everything you own, in one place." | Living benefit only; single line under "Anchora" |
| **og:title** (bold share headline) | "Anchora — Everything You Own, in One Place" | Short, emotional hook |
| **og:description** (share subtitle) | "Document every account and property you own today. Give one trusted person a clear, guided path to find it all. Free to start." | What + CTA |
| **Twitter title** | "Anchora — Everything You Own, in One Place" | Same as og:title |
| **Twitter description** | "Document every account and property you own today. Give one trusted person a clear, guided path to find it all." | Same as og:desc, trimmed |
| **SERP title** | "Anchora — Everything You Own, in One Place \| Nigeria" | Keyword-forward |
| **SERP description** | "Document your bank accounts, crypto, pensions, property, and more in one secure place. Designate a trusted person who is guided step-by-step through finding every record when the time comes. Free to start." | SEO keyword density + living benefit |

**Word removed:** "vault," "legacy," "wealth," "digital financial" from all shared surfaces. These stay in the product UI if already established but never appear in the first impression.

---

## Phase 1: Search Discovery (This Week — ~2 hours)

### 1.1 robots.txt

**File:** `public/robots.txt`

```
User-agent: *
Allow: /
Disallow: /api/
Disallow: /_next/
Sitemap: https://www.anchora.com.ng/sitemap.xml
```

Notes:
- `Disallow: /api/` keeps crawlers off backend routes
- `Disallow: /_next/` is belt-and-suspenders (Next.js blocks JS chunks anyway)
- You could also generate dynamically via `app/robots.ts` if you prefer

---

### 1.2 sitemap.xml

**Option A (recommended — static):** Install `next-sitemap`

```bash
npm install next-sitemap
```

Create `next-sitemap.config.js` at project root:

```js
/** @type {import('next-sitemap').IConfig} */
module.exports = {
  siteUrl: 'https://www.anchora.com.ng',
  generateRobotsTxt: false, // we already have a static one
  changefreq: 'weekly',
  priority: 0.7,
  exclude: ['/api/*', '/admin/*', '/dashboard/*', '/vault/*', '/settings/*'],
  transform: async (config, path) => {
    const priorities = {
      '/': 1.0,
      '/privacy': 0.3,
      '/terms': 0.3,
      '/security': 0.5,
    };
    return {
      loc: path,
      changefreq: path === '/' ? 'weekly' : 'monthly',
      priority: priorities[path] || config.priority,
      lastmod: new Date().toISOString(),
    };
  },
};
```

Add to `package.json` scripts:

```json
"postbuild": "next-sitemap"
```

This auto-generates `public/sitemap.xml` and `public/sitemap-0.xml` after every build.

**Option B (dynamic — no dependency):** Create `app/sitemap.ts`:

```ts
import { MetadataRoute } from 'next';

export default function sitemap(): MetadataRoute.Sitemap {
  const baseUrl = 'https://www.anchora.com.ng';

  return [
    {
      url: baseUrl,
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 1,
    },
    {
      url: `${baseUrl}/privacy`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.3,
    },
    {
      url: `${baseUrl}/terms`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.3,
    },
    {
      url: `${baseUrl}/security`,
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.5,
    },
  ];
}
```

---

### 1.3 Canonical URL

In your root layout (`app/layout.tsx`), add to the metadata export:

```ts
import type { Metadata } from 'next';

export const metadata: Metadata = {
  metadataBase: new URL('https://www.anchora.com.ng'),
  alternates: {
    canonical: '/',
  },
  // ... rest of your metadata
};
```

**Also set up www → non-www redirect** (or vice versa). If you're standardizing on `www.anchora.com.ng`, ensure `anchora.com.ng` 301-redirects to it. This is typically done at your hosting/DNS level (Vercel, Cloudflare, or your Nginx config).

---

### 1.4 JSON-LD Structured Data

Create `app/structured-data.tsx`:

```tsx
export function OrganizationSchema() {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Anchora',
    url: 'https://www.anchora.com.ng',
    logo: 'https://www.anchora.com.ng/images/logo-transparent.png',
    description:
      'Document your accounts and property in one secure place — designate a trusted person for guided discovery.',
    foundingDate: '2025',
    areaServed: {
      '@type': 'Country',
      name: 'Nigeria',
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

export function WebSiteSchema() {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'WebSite',
    name: 'Anchora',
    url: 'https://www.anchora.com.ng',
    inLanguage: 'en-NG',
    description:
      'Document your bank accounts, crypto, pensions, property, and more in one secure place. Designate a trusted person who is guided step-by-step through finding every record.',
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}

export function SoftwareApplicationSchema() {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'SoftwareApplication',
    name: 'Anchora',
    applicationCategory: 'FinanceApplication',
    operatingSystem: 'Web',
    url: 'https://www.anchora.com.ng',
    offers: {
      '@type': 'Offer',
      price: '0',
      priceCurrency: 'NGN',
      description: 'Free tier: up to 3 asset records',
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

In `app/layout.tsx`, add them inside `<head>`:

```tsx
import { OrganizationSchema, WebSiteSchema, SoftwareApplicationSchema } from './structured-data';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en-NG">
      <head>
        <OrganizationSchema />
        <WebSiteSchema />
        <SoftwareApplicationSchema />
      </head>
      <body>{children}</body>
    </html>
  );
}
```

Note: I set `lang="en-NG"` — this was `"en"` before. Nigerian English is a distinct locale that Google recognizes.

---

### 1.5 Fix the Empty Meta Tag

Next.js sometimes emits an empty `<meta>` tag. If you see it, add this to `app/layout.tsx` metadata:

```ts
export const metadata: Metadata = {
  // This suppresses the Next.js default empty viewport meta
  other: {
    // Forces a clean output
  },
  // ... rest
};
```

Or check if a layout component is rendering `<meta>` without content — that was the empty tag I found in the audit.

---

### 1.6 Final Metadata (Title, Description, Keywords, OG, Twitter)

This is the complete, approved metadata block — no "vault," "legacy," "wealth," or "digital financial." Paste into `app/layout.tsx`:

```ts
export const metadata: Metadata = {
  // ── Browser tab / SERP ──
  title: {
    default: 'Anchora — Everything You Own, in One Place | Nigeria',
    template: '%s | Anchora',
  },
  description:
    'Document your bank accounts, crypto, pensions, property, and more in one secure place. Designate a trusted person who is guided step-by-step through finding every record when the time comes. Free to start.',

  keywords: [
    'document assets Nigeria',
    'record keeping Nigeria',
    'asset documentation app',
    'trusted person guidance',
    'next of kin asset discovery',
    'bank account records Nigeria',
    'pension RSA discovery Nigeria',
    'crypto wallet records',
    'property document storage Nigeria',
    'what happens to my assets when I die Nigeria',
  ],

  metadataBase: new URL('https://www.anchora.com.ng'),
  alternates: { canonical: '/' },

  // ── Open Graph (WhatsApp, Facebook, LinkedIn) ──
  openGraph: {
    type: 'website',
    siteName: 'Anchora',
    title: 'Anchora — Everything You Own, in One Place',
    description:
      'Document every account and property you own today. Give one trusted person a clear, guided path to find it all. Free to start.',
    url: 'https://www.anchora.com.ng',
    images: [
      {
        url: '/images/og-default.png',
        width: 1200,
        height: 630,
        alt: 'Anchora — Everything you own, in one place.',
      },
    ],
  },

  // ── Twitter / X ──
  twitter: {
    card: 'summary_large_image',
    title: 'Anchora — Everything You Own, in One Place',
    description:
      'Document every account and property you own today. Give one trusted person a clear, guided path to find it all.',
    images: ['/images/og-default.png'],
  },

  // ── Crawling ──
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-video-preview': -1,
      'max-image-preview': 'large',
      'max-snippet': -1,
    },
  },
};
```

**Note on platform behavior:**
- **WhatsApp:** uses `og:title` + `og:description` + `og:image`. Your image has "Anchora" + "Everything you own, in one place." on it already, so the title + description add context without redundancy.
- **Twitter/X:** `summary_large_image` shows the image as a full-width card with title below it.
- **Google SERP:** uses the `title.default` and `description`, not OG tags.
```

---

### 1.7 OG Image (Photopea)

You're building this in Photopea. Specs:

| Element | Details |
|---------|---------|
| Canvas | 1200 × 630 px |
| Background | `#1E3A8A` (navy) |
| Logo icon | Your gold shield+keyhole+anchor PNG, centered, ~130px tall, ~120px from top |
| Brand name | "Anchora" — white serif (DM Serif Display, or Times New Roman), 72pt, centered |
| Tagline | "Everything you own, in one place." — white sans-serif (DM Sans, or Arial), 28pt, centered, ~16px below brand name |

That's it — two text elements, one logo. No accent line, no Nigeria bar, no subtitle. The platform-rendered title + description around the image do the selling; the image carries brand + emotional hook.

**Save as:** `public/images/og-default.png` → redeploy.

---

## Phase 2: Font Performance + Accessibility (~1 hour)

### 2.1 Font Preload `crossorigin`

Find your font preloads (these were in the `<head>`). If you're using `next/font`, Next.js handles this automatically — but the live site shows manual `<link rel="preload">` tags without `crossorigin`. If you're manually preloading, add it:

**Before:**
```html
<link rel="preload" href="/_next/static/media/1edc3ae7856af6c8-s.p.otf" as="font" />
```

**After:**
```html
<link rel="preload" href="/_next/static/media/1edc3ae7856af6c8-s.p.otf" as="font" crossorigin="anonymous" type="font/otf" />
```

If using `next/font`, remove the manual preloads and let Next.js handle them. Next.js automatically sets `crossorigin` and `type` correctly.

---

### 2.2 Skip-to-Content Link

Create `components/skip-link.tsx`:

```tsx
export function SkipLink() {
  return (
    <a
      href="#main-content"
      className="sr-only focus:not-sr-only focus:fixed focus:top-4 focus:left-4 focus:z-50 focus:px-4 focus:py-2 focus:bg-primary focus:text-white focus:rounded-md focus:outline-none"
    >
      Skip to main content
    </a>
  );
}
```

In your root layout, place it as the first child of `<body>` and add `id="main-content"` to your `<main>` element:

```tsx
<body>
  <SkipLink />
  <main id="main-content">{children}</main>
</body>
```

---

## Phase 3: Nigerian Pidgin Landing Page (3–4 hours)

### 3.1 Strategy

Create a Pidgin English variant of the landing page at `https://www.anchora.com.ng/pcm`. This is the single highest-ROI inclusivity move — Pidgin is understood by ~75M Nigerians, far more than formal English.

### 3.2 Route Setup

Create `app/pcm/page.tsx` (or `app/[locale]/page.tsx` if you prefer a proper i18n setup):

```tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Anchora — Everything You Own, in One Place (Nigerian Pidgin)',
  description:
    'Arrange where all your money and property dey. Put one trusted person wey go fit find everything when you no dey around.',
  alternates: {
    canonical: '/pcm',
  },
  openGraph: {
    locale: 'pcm_NG',
    // ... same og:image etc.
  },
};

export default function PidginPage() {
  // Reuse the same sections but with Pidgin copy
  return (
    <LandingPage locale="pcm" />
  );
}
```

### 3.3 Pidgin Copy (Key Sections)

Here's the full landing page in Nigerian Pidgin. Drop these strings into your component — same layout, different language.

```
# Hero
H1: "Your money and property need guardian wey no dey sleep"
Sub: "Anchora dey help you write down everything wey you get — bank accounts, crypto wallets, pension money, house documents, and more. We go show your trusted person step-by-step where everything dey when the time reach."

CTA: "Start free" / "See how e dey work"
Tagline: "Free to start · No need credit card"

# Problem Section
H2: "People dey lose money and property everyday"
Intro: "Ask yourself this small questions. If you no fit answer dem sharp-sharp, the person wey go come find your things go suffer am pass you."

Risk 1: "Anybody even sabi wetin you get?"
"Bank accounts, pensions, investments — no be just how to enter dem. Na whether dem even know say dem dey exist. By December 2024, Nigerian banks talk say almost 20 million accounts don sleep, plenty because nobody know where to look."

Risk 2: "Who dey responsible?"
"'My family go sort am out' no be plan. If nobody know say na dem get the work, everybody go think say anoder person don handle am."

Risk 3: "Dem go sabi how to collect dem?"
"Every bank, pension office, and government registry get their own process and plenty paper work. And the physical documents — house papers, car papers, certificates — many times dem only safe as the drawer or box wey dem dey inside."

# How It Works
H2: "Three steps to protect wetin you get"

Step 1: "Build your record"
"Write down all your money and property one by one — banks, crypto, investments, pension, house, car, and more. Everything get strong lock. Na you dey control wetin enter."

Step 2: "Pick your trusted person"
"Choose the one person wey you trust to handle the handover. Set the time wey you no go dey active (3, 6, 12, or 18 months). Anchora go guide your trusted person through finding everything when the time reach."

Step 3: "Dey active, dey in control"
"As you dey use your account normal, you still dey active — no separate check-in needed. If Anchora no hear from you for the time wey you set, e go start careful verification process before dem see anything."

# Asset Categories
H2: "Every account. One place."
Same category names, keep in English (Bank Account, Crypto Wallet, etc.) — these are proper nouns in the UI.

# Inactivity Engine
H2: "Careful, step-by-step release. Every time."

Step 1: "Active — Owner dey around. Vault lock tight."
Step 2: "Warning don go — We don send alert. Your inactive time don near."
Step 3: "Release don start — Grace period don finish, nobody respond."
Step 4: "Identity verified — Your trusted person don verify their identity."
Step 5: "Discovery dey guide — We dey show your verified trusted person step-by-step through each asset."

# Pricing
H2: "Simple, clear pricing"
Same pricing structure — numbers and tier names stay in English.

# CTA
H2: "Protect wetin be your own. Start today."
"To write down wetin you get no go take time. To find dem without you fit take years."

CTA: "Start free" / "Free to start · No need credit card"
```

### 3.4 Hreflang Tags

In your root layout, add hreflang links for both English and Pidgin:

```tsx
// In app/layout.tsx <head>
<link rel="alternate" hreflang="en-NG" href="https://www.anchora.com.ng" />
<link rel="alternate" hreflang="pcm-NG" href="https://www.anchora.com.ng/pcm" />
<link rel="alternate" hreflang="x-default" href="https://www.anchora.com.ng" />
```

---

## Phase 4: Blog (2–3 days)

### 4.1 Setup

Choose a path:
- **`next-mdx-remote`** (flexible, good for custom layouts)
- **Contentlayer** (type-safe, auto-generates types)
- **MDX with `@next/mdx`** (simplest, built-in Next.js support)

Simplest setup — `@next/mdx`:

```bash
npm install @next/mdx @mdx-js/loader @mdx-js/react
```

### 4.2 Initial 5 Blog Posts (SEO-First)

Target these Nigerian search intents (each ~1,500–2,500 words):

| # | Title | Target Keyword | Funnel Stage |
|---|-------|---------------|-------------|
| 1 | "What Happens to Your Nigerian Bank Account When You Die: The Complete Guide" | bank account after death Nigeria | Attention |
| 2 | "How to Document Your Assets in Nigeria: A Step-by-Step Guide" | document assets Nigeria | Interest |
| 3 | "Nigerian Pension (RSA) Next-of-Kin: What Your Family Needs to Know" | RSA next of kin Nigeria | Attention |
| 4 | "Digital Inheritance in Nigeria: What Happens to Your Crypto, Stocks, and Online Accounts" | crypto inheritance Nigeria | Attention |
| 5 | "5 Reasons Nigerian Families Fight Over Inheritance — And How to Prevent It" | inheritance dispute Nigeria | Attention |

Each article should:
- Target a specific long-tail keyword in the H1 and first 100 words
- Include at least 3 internal links to other blog posts
- End with a CTA: "Start documenting today — 3 records free"
- Include FAQPage structured data at the bottom

### 4.3 Blog Structured Data

Add this schema to every blog post:

```tsx
export function BlogPostingSchema({
  title,
  description,
  datePublished,
  authorName,
  url,
}: {
  title: string;
  description: string;
  datePublished: string;
  authorName: string;
  url: string;
}) {
  const schema = {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    headline: title,
    description,
    datePublished,
    author: {
      '@type': 'Person',
      name: authorName,
    },
    publisher: {
      '@type': 'Organization',
      name: 'Anchora',
      logo: {
        '@type': 'ImageObject',
        url: 'https://www.anchora.com.ng/images/logo-transparent.png',
      },
    },
    mainEntityOfPage: {
      '@type': 'WebPage',
      '@id': url,
    },
  };

  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
    />
  );
}
```

---

## Phase 5: Google Search Console + Analytics (30 min)

### 5.1 Google Search Console

1. Go to https://search.google.com/search-console
2. Add property: `anchora.com.ng` (domain property, includes www + non-www)
3. Verify via DNS TXT record (your domain provider) or HTML file upload
4. Submit `sitemap.xml` URL
5. Use the "URL Inspection" tool on the homepage to request indexing

### 5.2 Google Analytics (optional but useful)

GA4 tracking ID is all you need. Add to your layout if not already present.

### 5.3 Bing Webmaster Tools

Import from Google Search Console (Bing supports this). Bing is relevant for some Nigerian users.

---

## On-Page Checklist (Per Page)

Every page on anchora.com.ng should have:

- [ ] Unique `<title>` (50–60 chars, keyword near front)
- [ ] Unique `<meta name="description">` (120–158 chars, action-oriented)
- [ ] Exactly one `<h1>` containing the primary keyword
- [ ] Hierarchical headings (H1 → H2 → H3, no skips)
- [ ] `alt` text on every image (descriptive, includes keywords where natural)
- [ ] `lang="en-NG"` on `<html>`
- [ ] Canonical URL
- [ ] Open Graph + Twitter Card tags (with image)
- [ ] Internal links to 2–3 other pages on the domain
- [ ] Fast LCP (< 2.5s on mobile 3G) — check with Lighthouse

---

## Post-Deploy Verification Checklist

Run through these in order after every Phase 1 deploy. Each step takes < 5 minutes.

### 1. Static file checks (terminal)

```bash
# 1a. robots.txt — should return your rules, not 404
curl -s https://www.anchora.com.ng/robots.txt

# 1b. sitemap.xml — should return XML, not 404
curl -s https://www.anchora.com.ng/sitemap.xml | head -5

# 1c. canonical — should show www.anchora.com.ng
curl -s https://www.anchora.com.ng | grep -o 'rel="canonical" href="[^"]*"'

# 1d. structured data — should show 3 schemas in source
curl -s https://www.anchora.com.ng | grep 'application/ld+json' | wc -l
# Expected: 3
```

### 2. Structured data validation (browser)

1. Open https://search.google.com/test/rich-results
2. Enter: `https://www.anchora.com.ng`
3. Confirm all 3 schemas validate:
   - [ ] Organization (gold shield logo, "Document your accounts and property...")
   - [ ] WebSite (inLanguage: en-NG, description present)
   - [ ] SoftwareApplication (Free tier, NGN currency)

### 3. Google Search Console

1. Go to https://search.google.com/search-console
2. Add property: `anchora.com.ng` (domain property)
3. Verify via DNS TXT record at your domain registrar
4. Submit `sitemap.xml`
5. Use **URL Inspection** on homepage → **Request Indexing**

### 4. Social card test

1. Open https://www.opengraph.xyz and enter `anchora.com.ng`
2. Confirm the preview shows:
   - [ ] OG image (navy card with logo + "Everything you own, in one place.")
   - [ ] Title: "Anchora — Everything You Own, in One Place"
   - [ ] Description: "Document every account and property you own today..."

### 5. Live platform test

- [ ] Share the link in a **WhatsApp** chat — does the card preview render?
- [ ] Post on **Twitter/X** — does the large image card appear?
- [ ] Send in **iMessage** — does the rich preview show?

### 6. Redirect check

```bash
# Ensure non-www redirects to www (or vice versa, whichever you chose)
curl -sI https://anchora.com.ng | grep -i location
# Should show: location: https://www.anchora.com.ng (301)
```

### 7. Lighthouse

1. Chrome DevTools → Lighthouse → SEO category
2. Target: **100**

---

## Verification Steps (summary)

Quick curl commands after any deploy:

```bash
# 1. Robots
curl https://www.anchora.com.ng/robots.txt

# 2. Sitemap
curl https://www.anchora.com.ng/sitemap.xml | head -5

# 3. Canonical
curl -s https://www.anchora.com.ng | grep -o 'rel="canonical" href="[^"]*"'

# 4. Structured data
curl -s https://www.anchora.com.ng | grep 'application/ld+json'

# 5. Redirect
curl -sI https://anchora.com.ng | grep -i location

# 6. Rich Results validator
# Open: https://search.google.com/test/rich-results
# Enter: https://www.anchora.com.ng

# 7. Social card preview
# Open: https://www.opengraph.xyz
# Enter: https://www.anchora.com.ng

# 8. Lighthouse SEO score
# Chrome DevTools → Lighthouse → SEO → Target: 100
```

---

## Priority Order (What to Do When)

| Order | Task | Time | Impact |
|-------|------|------|--------|
| 1 | `robots.txt` | 15 min | 🔴 Unblocks crawling |
| 2 | `sitemap.xml` | 30 min | 🔴 Unblocks discovery |
| 3 | Canonical + full metadata block (vault-free copy) | 15 min | 🔴 SERP + social + duplicate prevention |
| 4 | JSON-LD schemas (3) | 30 min | 🟠 Rich results eligibility |
| 5 | OG image (Photopea — see 1.7 spec) | 30 min | 🟠 Social sharing renders |
| 6 | Post-deploy verification (checklist above) | 30 min | 🔴 Confirm everything works |
| 7 | Google Search Console + index request | 30 min | 🟠 Enters search index |
| 8 | Font crossorigin fix | 15 min | 🟢 Mobile LCP |
| 9 | Pidgin landing page | 3-4 hrs | 🔴 10× audience reach |
| 10 | Blog + 5 articles | 2-3 days | 🟡 Content flywheel |
| 11 | Hausa/Yoruba/Igbo pages | 2 weeks | 🟡 Full Nigerian coverage |