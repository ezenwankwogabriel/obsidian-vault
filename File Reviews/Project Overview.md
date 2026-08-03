---
type: project-overview
project: Obsidian Vault Setup
status: complete
started: 2026-08-02
completed: 2026-08-03
tags: [meta, workflow, setup]
---

# Project: Obsidian Vault Setup

**Goal:** Create a review surface for the Hermes Agent — the agent maps files it touches into a human-readable Obsidian vault, so the user can review offline.

## What was built

| Component | Detail |
|-----------|--------|
| **Vault** | `/root/obsidian-vault` (plain Markdown, valid Obsidian vault) |
| **Config** | `OBSIDIAN_VAULT_PATH` set in `~/.hermes/.env` |
| **Remote** | `github.com/ezenwankwogabriel/obsidian-vault` — credential stored repo-local in `.git/credentials` |
| **Skill** | `file-review` at `~/.hermes/skills/file-review/SKILL.md` — encodes the workflow; auto-creates notes and pushes |
| **Template** | `Templates/File Review.md` — frontmatter-driven note template |
| **Index** | `File Reviews/Index.md` — Dataview query + manual list |
| **Seed notes** | `pyproject.toml.md`, `AGENTS.md.md`, `run_agent.py.md`, `model_tools.py.md`, `cli.py.md` |

## Architecture choice

- **Hermes Projects** (`project_create` / `project_switch`) handle active context isolation per work initiative.
- **Obsidian vault** handles durable, human-readable knowledge that outlives any one conversation or project.
- **`file-review` skill** bridges the two: agent writes notes during work, vault persists them after.

## Vault conventions
- One note per file under `File Reviews/<basename>.md`
- Frontmatter: `type: file-review`, `module`, `path`, `status` (needs-review|reviewed|flagged), `tags`
- Sections: Why touched, What it does, Key bits, Open questions, Linked files
- After writing: `git add -A && git commit && git push origin main`

## Files mapped (so far)
- [[pyproject.toml]] — build/deps manifest (exact pins, Python cap)
- [[AGENTS.md]] — dev guide, contribution rubric, footprint ladder
- [[run_agent.py]] — AIAgent class (7,551 lines, thin forwarder)
- [[model_tools.py]] — tool orchestration (discovery, dispatch, async bridging)
- [[cli.py]] — interactive CLI / TUI (18,394 lines, 3 mixins)
- [[file-review skill]] — meta note about the skill itself

## What was NOT done (parked)
- **Honcho integration** — deferred; the user wasn't sure yet
- **Auto-merge / bidirectional sync** — deliberately one-directional (vault is review surface, not source of truth)
- **Obsidian Git plugin setup** — user needs to install and configure on their local machine

## How to resume this project later
```
/project switch "obsidian-vault-setup"
```
Then read this note — it covers everything that was set up. New file mappings use the same `file-review` skill.