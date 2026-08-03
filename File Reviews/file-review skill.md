---
type: file-review
module: meta
path: /root/.hermes/skills/file-review/SKILL.md
purpose: Agent skill that encodes the file-review → Obsidian workflow. Created as part of the initial vault setup.
status: needs-review
reviewed_on: 2026-08-03
tags: [meta, workflow]
---

# file-review skill

**Path:** `/root/.hermes/skills/file-review/SKILL.md`
**Module:** meta
**Purpose:** Agent skill that encodes the file-review → Obsidian workflow — creates standardized review notes and pushes them to GitHub for local Obsidian consumption.
**Status:** needs-review
**Reviewed:** 2026-08-03

## Why the agent touched it
To confirm the vault setup, credential storage, and git sync pipeline work end-to-end.

## What it does
- Defines when to create review notes (user asks, multi-step changes, non-obvious constraints).
- Specifies the note format (YAML frontmatter + markdown sections).
- Instructs the agent to commit-and-push after each note so the user can pull into local Obsidian.

## Key bits / constraints
- Vault path is read from `OBSIDIAN_VAULT_PATH` in `~/.hermes/.env`.
- Remote is `https://github.com/ezenwankwogabriel/obsidian-vault.git`.
- Credentials stored repo-local in `.git/credentials` (chmod 600), never committed.
- Notes go in `File Reviews/<basename>.md`.

## Open questions & flags
- Index.md includes both a Dataview query and a manual list; the manual list will drift if the agent forgets to update it. Consider dropping the manual section and relying purely on Dataview once confirmed.

## Linked files
- [[AGENTS.md]] — the skill mirrors AGENTS.md's "CLI command + skill" rung of the Footprint Ladder.