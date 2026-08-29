# If Big Tech AI Solved the Anchora Problem

*Strategic thought exercise — August 2026. What would Google, Apple, or OpenAI build if they targeted digital legacy?*

---

## The Core Difference

Anchora's model: **User documents → inactivity timer → release to Trusted Contact.**

A top AI company would invert this entirely: **AI discovers → AI verifies → AI guides.** The user does almost nothing. The system is ambient.

---

## Product: "Continuum" (Representative Name)

Not a vault. Not a record-keeping tool. A **digital continuity fabric** that runs silently on-device and in the background of your digital life.

### What It Actually Does

| Layer | What | How |
|-------|------|-----|
| **Passive Asset Discovery** | Automatically builds and maintains a living map of every account, asset, policy, and holding | Parses email receipts, banking notifications, brokerage statements, crypto transactions, property tax emails — constructs a graph |
| **Behavioral Liveness Verification** | Continuously confirms you're still you, actively alive and in control | Behavioral biometrics: typing cadence, scroll patterns, phone tilt, gait from IMU sensors, app-open rhythms, Face ID/Touch ID unlock cadence |
| **Deviation Detection** | Distinguishes "on vacation" from "incapacitated" from "deceased" | ML model trained on your personal baseline; cross-referenced with life signals (messages sent, purchases made, locations visited, biometric unlocks) |
| **Progressive Handover** | No binary release. Staged, reversible, confidence-gated | Gentle check-in → Trusted Contact notified → read-only browse → guided institutional discovery → full access |

### The User Experience

The user opens the app once, designates Trusted Contacts, and **never opens it again**. The system runs for decades without interaction. That's the product — not a dashboard, not a vault interface. Invisibility is the feature.

---

## Technology Stack

### 1. On-Device Behavioral ML (THE MOAT)

```
Phone sensors → Local ML model → Anomaly score → Cloud (encrypted)
                                     ↑
                              Never leaves device
```

- **Typing dynamics**: key hold time, flight time between keys, error patterns, autocorrect acceptance rate
- **Touch biomechanics**: pressure curves, contact area, swipe velocity/acceleration, pinch geometry
- **Device handling**: gyroscope/accelerometer signatures while reading, walking, lying down
- **App rhythms**: which apps opened in what order, time of day patterns, session duration distributions
- **Biometric cadence**: Face ID / Touch ID unlock frequency and timing patterns

The raw sensor data **never leaves the device**. Only a daily encrypted liveness attestation is sent to cloud. This is both a privacy feature and a defensibility lock — no competitor can replicate this without OS-level sensor access.

### 2. Asset Graph Engine

An AI pipeline that:
1. Scans your email (with permission) for financial statements, receipts, confirmations
2. Extracts structured data: institution name, account type, approximate value, last activity date
3. Builds a **living graph** of your financial footprint
4. Refreshes as new emails arrive
5. Flags dormant accounts, forgotten 401(k)s from old employers, unused crypto wallets

This only works for a company with email access (Google) or deep OS integration (Apple reading Mail.app + notifications).

### 3. Federated Liveness Model

- Each user's behavioral model is **personal** — trained on their own data
- Base model ships pre-trained on anonymized population data (federated learning across millions of devices)
- Personalizes within days, reaches high confidence within weeks
- Detects: vacation (behavioral shift + location change = low alert), depression/illness (slower, less varied patterns = medium alert), death/coma (complete cessation of all signals = high alert after calibrated period)

### 4. Secure Handover Cryptography

- Trusted Contacts' keys are held in a **threshold scheme** — no single party (including the company) can decrypt without the release condition being met
- Release requires: behavioral liveness below threshold AND timeout period elapsed AND (optional) external signal (death certificate API, obituary detection)
- Zero-knowledge proofs: the system proves to Trusted Contacts that conditions were met without revealing the behavioral data that triggered it

---

## Business Model

### B2B2C, Not D2C

Direct-to-consumer death-planning apps have terrible conversion. Instead:

| Channel | Product | Revenue |
|---------|---------|---------|
| **Banks** | White-label "Digital Continuity" as premium account feature | Per-user SaaS fee |
| **Insurers** | Bundled with life insurance; reduces unclaimed policy problem | Revenue share on recovered policies |
| **Employers** | Employee benefit: "your family never loses access" | Per-employee/month |
| **Platforms** | API for social media, crypto exchanges, cloud providers to handle deceased users | Transaction/verification fee |

### Consumer Tier

- Free: 1 Trusted Contact, basic asset discovery (email only), 30-day inactivity window
- Premium ($9.99/mo or $99/yr): 5 Trusted Contacts, full asset graph, behavioral verification, crypto wallet discovery, legal document integration

---

## Moat Analysis

### Unbreachable Moats (Only Big Tech Has These)

| Moat | Why Only Them |
|------|---------------|
| **OS-level sensor access** | Apple/Google control the IMU, keyboard, touchscreen APIs. Third-party apps can't get raw sensor data in background. |
| **Email corpus** | Google has 15+ years of Gmail for billions of users. The asset graph training data is irreplaceable. |
| **Behavioral dataset** | Billions of device-years of typing, scrolling, walking patterns. Federated training at population scale. |
| **Ecosystem lock-in** | If iCloud/Google Account IS the product, users can't leave without losing everything. The continuity feature becomes a retention mechanism, not just a product. |
| **Death certificate API access** | Government integrations at scale — only companies with legal/compliance armies can negotiate jurisdiction-by-jurisdiction |

### What Anchora Has That They Don't

| Anchora's Edge | Why It Matters |
|----------------|----------------|
| **Nigerian institutional knowledge** | PFA/PenCom procedures, bank next-of-kin workflows, NGX/CHN identifiers, BVN/NIN integration. Big Tech doesn't understand Nigerian bureaucracy. |
| **QoreID integration** | Local identity verification that actually works in Nigeria. Google/Apple would need years to build local compliance. |
| **Market-specific Trusted Contact model** | Nigerian family structures, communal asset ownership patterns, informal holdings — Big Tech models would be trained on Western nuclear-family assumptions |
| **Speed of local iteration** | Can ship Nigeria-specific features in weeks vs. Big Tech's multi-year global rollout cycles |
| **Paystack-native billing** | No Stripe. Local payment rails. |

---

## The Strategic Insight

Big Tech AI would build the *opposite* of Anchora:
- **Passive** (AI discovers assets) vs. **Active** (user documents assets)
- **Behavioral verification** (AI detects death) vs. **Timer-based** (30-day inactivity)
- **Global horizontal** (works everywhere, shallowly) vs. **Local vertical** (Nigeria, deeply)
- **OS-embedded** (invisible feature) vs. **Standalone SaaS** (visible product)

Anchora's defensible position is NOT the technology — it's the **Nigerian institutional process knowledge** that no Silicon Valley company will invest in acquiring. The entire product is a moat made of NIN, BVN, PenCom, QoreID, and knowing which bank requires a notarized affidavit vs. which one accepts a next-of-kin form.

**Anchora wins by being local. Big Tech wins by being ambient. These are different games.**

---

## What Anchora Should Steal

1. **Progressive handover** — staged release, not binary. Gentle check-in → notification → browse → full access
2. **Asset discovery hints** — not full email parsing, but smart suggestions: "We noticed you haven't added any pension accounts. Nigerian workers typically have a PFA. Want to add yours?"
3. **Liveness check-ins** — periodic "Are you still there?" with escalating urgency, not a silent timer
4. **B2B2C distribution** — partner with PFAs, banks, insurers as distribution, not organic acquisition