# Anchora — Digital Financial Legacy Vault

## Quick Reference

| Field | Value |
|-------|-------|
| Domain | anchora.com.ng |
| Founder/Gabby | Gabriel (user of this assistant) |
| Builder | Claude Code on Gabby's machine |
| Market | Nigeria |
| Category | Record-keeping & guided-discovery reference tool |
| Frontend | Next.js 14 (App Router), TypeScript, Tailwind, shadcn/ui |
| Backend | Railway (Postgres), Cloudflare R2, Redis, Paystack |
| ID Verification | QoreID (Trusted Contact only) |
| Encryption | Field-level at rest (vault data) |
| Sprint (current) | Sprint 9 — Identity verification (QoreID) |
| Sprint (next) | Sprint 10 — Admin Dashboard |

## Core Positioning (Non-Negotiable)

Anchora is a **Nigerian-market SaaS record-keeping and guided-discovery reference tool**. Account Owners document where their financial and physical assets are held. A single designated **Trusted Contact** per account receives guided institutional discovery support after the account holder's prolonged inactivity.

**Anchora is explicitly NOT:**
- An estate manager or estate planning platform
- A credential vault (does not store login credentials)
- A financial institution, wealth manager, or asset custodian
- A facilitator of direct asset transfer

### Compliance Rules (Apply to All Future Work)
- No "estate," "recovery" (user-facing), "beneficiary," "executor," "manage," or credential-handover language in user-facing copy
- No implication of financial planning, wealth management, asset appraisal, or outcome guarantees
- Correct user-facing label: **"Trusted Contact"** (or "Trusted Person")
- Release-side language moving from "recovery" → "discovery"/"guidance" (in progress)
- Single Trusted Contact per account (current model; multi-contact decided but not yet built)
- Identity verification never framed as owner mandate, never uses "estate" language

> Backend internals (routes like `/executor/*`, schema fields `notifiedAt`/`emailVerifiedAt`) intentionally keep old naming — renames are copy-layer only unless explicitly scoped.

## User Roles

- **Account Owner** — creates/manages vault, designates Trusted Contact
- **Trusted Contact** — one per owner. Gains guided institutional discovery after release is triggered (prolonged inactivity). Must pass QoreID identity verification before accessing released records.
- **Administrator** — internal dashboard (READ_ONLY / ADMIN / SUPER_ADMIN), separate Zustand store + JWT

## Tech Stack & Architecture

### Frontend
- Next.js 14 (App Router), TypeScript throughout
- Tailwind CSS, shadcn/ui (unstyled, accessible — owned/customized)
- React Hook Form + Zod for forms
- Zustand for state management
- Lucide React icons
- DM Serif Display (headings), DM Sans (body), 16px base

### Backend
- Railway (Postgres) — daily `pg_dump` → gzip → R2 backup via rclone
- Cloudflare R2 — document storage (zero egress, short-lived access URLs)
- Redis — activity tracking at token refresh layer only
- Paystack — payments (locked provider, no alternative gateway abstraction)
- QoreID — identity verification via `@qore-id/web-sdk`, server-side session proxy
- Field-level encryption at rest for vault data (including `estimatedValue` in kobo)

### Design System
- Light theme only (MVP; no dark mode)
- Navy + blue palette: `#1E3A8A` primary, `#3B82F6` accent, `#10B981` mint/success
- Buttons `rounded-lg` (8px), cards `rounded-xl` (12px) + `shadow-sm`, inputs `rounded-md` with blue focus ring
- Card-based layouts, step-by-step guided flows, color-coded status strips
- No em dashes in any copy strings

## Vault Records

- **11 flat asset categories** (Financial & Digital Accounts, Physical Assets, Other)
- `getFieldConfig(category)` — single source of truth for field labels/placeholders per category
- `isPhysicalCategory(category)` — drives physical-specific UI branching
- `FREE_TIER_RECORD_LIMIT` — single constant for paywall, client-side checks, downgrade lock
- Optional `estimatedValue` (kobo, encrypted) — decided, not yet built (see Roadmap)

## Billing Model

| Tier | Records | Notes |
|------|---------|-------|
| FREE | Limited (3) | Current live tier |
| PRO | Unlimited | "Coming soon" — blocked on Paystack live-mode approval |

- Annual discount percentage finalized (not a placeholder)
- Frontend never optimistically marks user PRO — only updates after Paystack webhook confirms
- Downgrade/cancellation: read-only access above limit until `currentPeriodEnd`, then locked (not deleted)
- Open: Paystack Inline JS vs. hosted redirect (not blocking launch)

## Sprint Status

### Complete (Sprints 1–8)
Scaffold, auth, onboarding/dashboard, vault, Trusted Contact rename/management, settings, billing, release flow work.

### Current: Sprint 9 — Identity Verification (QoreID)
- Session token minted server-side via backend proxy (`POST /v1/sessions`) — credentials stay server-side
- Verification status confirmed server-side only (never set optimistically from client-side QoreID `success` event)
- Gate applies to **Trusted Contact only** (account owners not required to verify)
- Verification UI is generic, lives in Settings, one-time account-level action

### Next: Sprint 10 — Admin Dashboard
- Roles: READ_ONLY, ADMIN, SUPER_ADMIN
- Hard delete UI intentionally omitted for MVP

## Decided-but-Not-Built (Roadmap)

1. **Release-side language audit**: "recovery" → "discovery"/"guidance" across remaining surfaces
2. **Multi-Trusted-Contact (1-of-N)**: up to 5 contacts on Premium. Any verified contact gets guided discovery access. See [[Projects/Anchora - Multi-Trusted Contact Spec]] for full implementation spec
3. **Documented Value feature**: optional `estimatedValue` on all 11 categories; dashboard card; batch "Update values" modal. No live price lookups, staleness, trends, or snapshot model.
4. **Marketing site correction**: anchora.com.ng still describes superseded credential-vault model — needs alignment with record-keeping/reference-tool positioning

## Deferred (Post-MVP)
- B2B/employer tier
- WhatsApp/SMS notifications
- Physical asset UI complexity beyond basic CRUD
- Secondary `.com` domain for redirects

## Working Principles

- **Decision-first, then implementation prompt** — product/design decisions made in conversation, then distilled into tightly scoped build prompts (intent-and-constraint framing, not prescriptive className instructions)
- **Sequential, not batched** — one discrete change or sprint at a time
- **Read-only audit before correction pass** for compliance/copy work
- **Scope discipline** — out-of-scope items explicitly named and parked
- **No em dashes** in any copy or prompt text
- **Don't add infrastructure for unmeasured behavior** (retracted ValueSnapshot/trend chart is cautionary tale)
- Defensible differentiation: Nigerian institutional process knowledge (PFA/PenCom, bank next-of-kin procedures, NGX/CHN identifiers)
- B2B2C partnerships (banks, insurers, PFAs, employers) are likely scale channel, not organic search

## Collaboration Model

Gabby produces architectural specs/design docs into this shared workspace (Obsidian vault); Gabby pulls them into Claude Code on their machine for implementation and reports back progress.

**Status**: active development, Sprint 9 in progress.

[[File Reviews/Index|Vault Index]]