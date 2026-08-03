---
type: file-review
module: build
path: /usr/local/lib/hermes-agent/pyproject.toml
purpose: Project manifest — package name, Python bounds, and the exact-pinned core dependency set.
status: needs-review
reviewed_on: 2026-08-02
tags: [hermes, build, dependencies, supply-chain]
---

# pyproject.toml

**Path:** `/usr/local/lib/hermes-agent/pyproject.toml`
**Module:** build
**Purpose:** Project manifest — package name, Python bounds, and the exact-pinned core dependency set.
**Status:** needs-review
**Reviewed:** 2026-08-02

## Why the agent touched it
Read while standing up the Obsidian review vault, to ground the demo note in a real file and confirm the codebase identity/version.

## What it does
- Declares the `hermes-agent` package, current version `0.19.1`.
- Pins every direct dependency to an exact `==X.Y.Z` (no ranges).
- Encodes the `requires-python = ">=3.11,<3.14"` cap and *why* it exists.

## Key bits / constraints
- **Python ceiling is load-bearing.** Capped `<3.14` because Rust-backed transitives (e.g. pydantic-core) have no cp314 wheel yet; otherwise uv would fall back to a maturin source build that fails. Raise the ceiling only after those wheels ship.
- **Exact pins are deliberate.** Ranges would let PyPI ship fresh transitive versions unreviewed. Bumping a pin requires also regenerating `uv.lock` (`uv lock`). This tightening followed the "Mini Shai-Hulud" PyPI worm hitting `mistralai` 2.4.6.
- **Scope rule for `dependencies`:** only packages used by *every* hermes session belong here. Provider-specific libs (`anthropic`, `firecrawl-py`, `fal-client`, `edge-tts`, …) live in extras and are lazy-installed via `tools/lazy_deps.py`. Smaller `dependencies` = smaller supply-chain blast radius.
- **Two version bumps carry specific fixes:**
  - `requests==2.33.0` — pulls in CVE-2026-25645 fix.
  - `pydantic==2.13.4` (bumped from 2.12.5) — pulls pydantic-core 2.46.4, which fixes a segfault when the OpenAI SDK Responses API resource is exercised from a non-main thread (the `codex_responses` dispatch).

## Open questions & flags
- Is `<3.14` still the right ceiling? Check whether cp314 wheels exist for pydantic-core / other Rust transitives before raising it.
- The `dependencies` block is large; confirm no entry has drifted into "only used by one backend" territory and should move to an extra.

## Linked files
- [[AGENTS.md]] — explains the "narrow waist" philosophy that motivates the exact-pin + extra split.
