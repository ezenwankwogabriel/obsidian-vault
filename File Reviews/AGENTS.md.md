---
type: file-review
module: docs
path: /usr/local/lib/hermes-agent/AGENTS.md
purpose: Development guide for AI assistants/contributors — intent layer, contribution rubric, and the Footprint Ladder for adding capability.
status: needs-review
reviewed_on: 2026-08-02
tags: [hermes, contributing, architecture, conventions]
---

# AGENTS.md

**Path:** `/usr/local/lib/hermes-agent/AGENTS.md`
**Module:** docs
**Purpose:** Development guide for AI assistants/contributors — intent layer, contribution rubric, and the Footprint Ladder for adding capability.
**Status:** needs-review
**Reviewed:** 2026-08-02

## Why the agent touched it
Read to understand the project's design intent before proposing any Obsidian/Honcho integration, and to mirror its conventions in the review-vault demo notes.

## What it does
- Documents what Hermes *is* (one agent core across CLI/TUI/gateway/desktop; capability at the edges via plugins/skills).
- States two lens-shaping properties: **per-conversation prompt caching is sacred** and **the core is a narrow waist**.
- Gives a "Contribution Rubric" (what we want / what we reject) and a "Footprint Ladder" for where new capability should live.

## Key bits / constraints
- **Prompt caching is sacred.** Anything that mutates past context / swaps toolsets / rebuilds the system prompt mid-conversation invalidates the cache and multiplies user cost.
- **Footprint Ladder (cheapest first):** extend existing code → CLI command + skill → service-gated tool (`check_fn`) → plugin → MCP server → new core tool (last resort). New core tools ship on every API call, so the bar is high.
- **`.env` is for secrets only** (API keys/tokens/passwords). Behavioral settings (timeouts, flags, prefs) go in `config.yaml`. Don't tell users to "set X in .env" unless X is a credential.
- **Third-party products don't land in the core tree.** Vendor SaaS / observability / analytics integrations ship as standalone plugin repos under `~/.hermes/plugins/`, not under `plugins/` in this repo.
- **Verify the premise before fixing.** Many rejected PRs fix a "gap" that is actually intentional design. Trace real code (`git log -p -S`) before assuming something is unfinished.
- **No lazy-reading pagination** on instructional tools (skills/prompts/playbooks) — models read page 1 and skip the rest.
- **File dependency chain:** `tools/registry.py` (no deps) ← `tools/*.py` (register at import) ← `model_tools.py` ← `run_agent.py`/`cli.py`/etc.

## Open questions & flags
- Tension with the `obsidian` skill: that skill reads `OBSIDIAN_VAULT_PATH` from `~/.hermes/.env`, but AGENTS.md says non-secret config belongs in `config.yaml`. The vault path is a path, not a secret — acceptable, but worth noting. If you want strict compliance, move the path into `config.yaml` and patch the skill to read it there.
- The doc is ~1435 lines and truncated in the read; the later sections (CLI command registry, TUI, Electron desktop, plugin ABCs) were not fully reviewed.

## Linked files
- [[pyproject.toml]] — the exact-pin + extra split this doc's "narrow waist" philosophy justifies.
