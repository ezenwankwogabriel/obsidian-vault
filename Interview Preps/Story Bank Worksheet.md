# Story Bank Worksheet

## How to build it

1. The 5 stories below are **drafted from your real experience** — review, correct the facts, and tighten the language.
2. Fill the remaining 3–7 rows with additional stories from your own history.
3. For each, map it to the competencies it can serve (aim for ≥3 per story).
4. Quantify every Result. If you can't put a number on it, you haven't finished the story.

**Rule of thumb:** 8 solid stories, well-mapped, covers ~95% of behavioral questions you'll face.

---

## 1. The Story Grid

| # | Story (one-line label) | Competencies it serves |
|---|---|---|
| 1 | Redesigned Anchora's trusted-contact model (primary/backup → broadcast) | **CH** · LE · AD · CO |
| 2 | Positioned Anchora correctly — not an estate manager, not a credential vault | **CH** · LE · AD |
| 3 | Fixed the inactivity-release pipeline bugs and documented the system | **CH** · TE |
| 4 | Built content strategy + delivery system from zero (no team, no budget) | LE · AD · PR |
| 5 | Self-directed Go + backend architecture learning while building a startup | AD · PR · CH |
| 6 |  |  |
| 7 |  |  |
| 8 |  |  |
| 9 |  |  |
| 10 |  |  |
| 11 |  |  |
| 12 |  |  |

**Competency shorthand:** **CH**allenge · **FA**ilure · **CO**nflict · **LE**adership · **TE**amwork · **AD**aptability · **PR**ioritization · **AC**hievement

---

## 2. Drafted Stories (review, correct facts, polish)

### Story 1 — Redesigned the trusted-contact model

**Competencies:** Challenge, Leadership, Adaptability, Conflict
**Target questions:** "Tell me about a challenge you resolved," "Tell me about a time you led," "Tell me about a time you had to make a decision with incomplete information"

> **Situation:** While building Anchora's trusted-contact feature, I discovered the original plan assumed a primary/backup model — one contact with a fallback. For a product whose core promise is "your assets reliably reach someone you trust," that backup was a single point of failure.
>
> **Task:** I needed to redesign the contact model so a user's inventory reliably reached trusted people, without over-complicating the product or introducing race conditions.
>
> **Action:** I mapped every failure case of the primary/backup model — what happens if the primary is unresponsive, if the backup changes emails, if the owner downgrades from Premium. I wrote a full implementation spec for a **broadcast model**: all verified contacts receive the asset inventory simultaneously — no voting, no race, no sequential activation. I resolved the tier limits (Free: 1 contact, Premium: up to 5), designed the invite-and-verify flow around QoreID identity verification, wrote the migration plan from the old single-contact schema, and addressed 10 edge cases including downgrade, contact removal during release, and unresponsive contacts.
>
> **Result:** The spec was approved and queued for implementation post-Sprint 9. It eliminates the single point of failure entirely — every verified contact gets the same inventory at the same time. Free users retain 1 contact, Premium expanded to up to 5.
>
> **Learning:** I learned to recognize when a "simple" assumption hides a reliability risk, and to resolve product ambiguity with a written spec *before* writing code.

---

### Story 2 — Corrected Anchora's positioning (not an estate manager)

**Competencies:** Challenge, Leadership, Adaptability
**Target questions:** "Tell me about a time you had to change course quickly," "Tell me about a time you pushed back on a decision," "Tell me about a time you led"

> **Situation:** Early on, Anchora's marketing copy and domain (anchora.com.ng) described us as an "estate management" and "credential vault" platform. When I audited this against what we actually built, there was a gap — we're a record-keeping and guided-discovery tool, not an estate manager — and the wrong positioning created legal risk and user confusion in the Nigerian market.
>
> **Task:** I needed to define Anchora's non-negotiable positioning and enforce it across every surface: copy, product, and future feature decisions.
>
> **Action:** I wrote the core positioning into our project master doc: Anchora is "a Nigerian-market SaaS record-keeping and guided-discovery reference tool." I listed what we are explicitly NOT — no estate manager, no credential vault, no financial institution — and banned specific words from user-facing copy: estate, beneficiary, executor, recovery, credential handover. I established the correct label ("Trusted Contact"), applied the rules retroactively to existing copy, and made compliance a gate for all future product work.
>
> **Result:** Zero positioning drift since. The GTM plan now leads with the "living value dashboard" — organize where everything is held, today — with the legacy question as the reveal, not the headline. Every sprint, marketing post, and feature decision is checked against these rules.
>
> **Learning:** In a regulated-adjacent market like Nigeria, your product category label isn't marketing — it's a legal and trust boundary. Defining it early and enforcing it strictly prevents far costlier corrections later.

---

### Story 3 — Fixed the inactivity-release pipeline

**Competencies:** Challenge, Teamwork (multi-agent review)
**Target questions:** "Tell me about a technical challenge you solved," "Tell me about a time you debugged a complex system"

> **Situation:** Anchora's inactivity-to-release pipeline — the subsystem that detects prolonged inactivity and releases asset records to trusted contacts — had correctness bugs that a multi-round review (CodeRabbit findings → multi-agent code review → manual follow-up) surfaced in the state machine, executor soft-delete logic, and notification timing.
>
> **Task:** I needed to fix the bugs and produce a single source of truth for the pipeline so future work wouldn't reintroduce the same issues.
>
> **Action:** I worked through the fixes systematically: converted executor removal from hard-delete to soft-delete with `removedAt` filtering, fixed a case-insensitive email uniqueness constraint, corrected a `notifyExecutor` bug where the notification was persisted *before* the send (failing sends left dangling notifications), added a `RELEASING` retry branch to the state machine, and extracted shared helpers to reduce duplication. After the fixes, I wrote a complete pipeline document covering the state machine (ACTIVE → NOTIFIED → RELEASING), the full `initiateRelease` walkthrough, the notification/reminder inventory, and — critically — a known-gaps section flagging an advisory-lock race, missing post-release reminders, and dead/unwired code.
>
> **Result:** All correctness bugs resolved. The pipeline document now lives alongside the code as the single source of truth, and the known-gaps section gives any future engineer a map of what's been thought through but not yet built.
>
> **Learning:** I learned that in a system where a bug means a dead person's family never gets their records, "it compiles" isn't enough — you need documented edge cases and a written failure-mode map.

---

### Story 4 — Built content strategy + delivery system from zero

**Competencies:** Leadership, Prioritization, Adaptability
**Target questions:** "Tell me about a time you took initiative," "Tell me about a time you built something from scratch," "Tell me about a time you had to prioritize"

> **Situation:** Anchora had no marketing presence, no content team, and no design budget. As a solo founder, I needed to build pre-launch awareness in the Nigerian market using text-only content — while also writing code and running the company.
>
> **Task:** I needed a content strategy that built trust and drove awareness without a team, a budget, or a live product, and a delivery system that didn't add drafting overhead to my daily work.
>
> **Action:** I designed a 4-pillar content strategy: Living Value Dashboard (35% — the present-tense benefit of organizing where everything is held), Problem Education (25% — making people feel the pain of not having their records), Founder Building (25% — building trust through transparency), and Product Updates (15%). I chose X/Twitter as the channel and set a 3-posts-per-week cadence. I then built a cron delivery system via my AI assistant that generates ready-to-post threads every 2 days, each labeled by pillar and phase, with compliance guardrails baked in (lead with living benefit, no "estate"/"beneficiary" language, product truth only).
>
> **Result:** Posts ship on cadence without manual drafting overhead. The system is live — I review the draft, post it, and spend 10 minutes engaging with replies. The content leads with the "where is everything, actually?" living-benefit framing, which early feedback confirms resonates better than death/estate framing.
>
> **Learning:** [FILL]

---

### Story 5 — Self-directed Go + backend learning while building a startup

**Competencies:** Adaptability, Prioritization, Challenge
**Target questions:** "Tell me about a time you learned something new fast," "Tell me about a time you had competing priorities"

> **Situation:** I'm building Anchora solo while also needing to grow as an engineer — specifically, I needed stronger backend fundamentals in Go and database architecture for the kind of reliability a financial vault product demands.
>
> **Task:** I needed to structure two parallel self-directed learning paths without sacrificing Anchora development velocity.
>
> **Action:** I built a 12-week Go learning plan — Tour of Go (3 days) → Learn Go with Tests (6-8 weeks, TDD-first through 25 chapters) → Gophercises applied projects → reference depth — and a parallel 16-week backend/database architecture path. I set up daily 60-90 minute blocks, a progress tracker with chapter-level check-ins, and the discipline to split my day: Anchora development during peak hours, learning in the margins. I completed Phase 1 in 3 days and am [FILL: chapters-done] chapters deep in Phase 2, writing tests-driven Go code alongside building a Next.js/Postgres production app.
>
> **Result:** In ~[FILL: weeks] weeks, I completed the Go Tour and 12 of 25 Learn Go with Tests chapters — including concurrency, mocking, and dependency injection — while keeping Anchora Sprint 9 (QoreID identity integration) on track. Neither path has blocked the other.
>
> **Learning:** [FILL]

---

## 3. More seeds to mine

Already-drafted stories above came from these seeds. Additional threads from your experience:

- **Conflict / disagreement** — any time you disagreed with a teammate, boss, or client about direction
- **Failure** — any missed deadline, failed approach, or launch that didn't land. (Founder stories are gold here — what did you try that didn't work before the current approach?)
- **Mentoring / teaching** — any time you helped someone learn or onboarded them
- **Achievement / proudest moment** — [FILL: what's the single thing you're most proud of?]

**For each, the two questions that turn it into a story:**
1. What was the *hardest moment* in that thread?
2. What *number* proves the outcome?

---

## 4. Polish checklist

- [ ] Situation is ≤2 sentences
- [ ] Every action sentence starts with "I", not "we"
- [ ] Result has at least one number/date/%
- [ ] Learning sentence is ready for challenge/failure questions
- [ ] Story runs 90–120 seconds when spoken
- [ ] Same story can flex to ≥2 different questions