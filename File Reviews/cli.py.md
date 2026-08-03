---
type: file-review
file: cli.py
module: hermes-agent (CLI orchestrator)
status: needs-review
reviewed_on: 2026-08-03
tags: [hermes-agent, cli, repl, tui, prompt-toolkit]
---

# File Review: `cli.py`

**Location:** `/usr/local/lib/hermes-agent/cli.py`
**Total lines:** 18,394
**Role:** The full-featured interactive CLI / TUI for Hermes Agent — REPL with `prompt_toolkit`, slash commands, rich formatting, toolset selection, and session management.

---

## What It Does

- Provides an interactive REPL (Read-Eval-Print Loop) using `prompt_toolkit` for a fixed input area TUI at the bottom of the terminal, with scrollable output above.
- `HermesCLI` class (line 4195) inherits from three mixins:
  - `CLIAgentSetupMixin` — agent initialization
  - `CLICommandsMixin` — slash command implementations
  - `CLIBillingMixin` — billing/cost display
- `process_command()` (line 9787) — dispatches slash commands (`/help`, `/quit`, `/profile`, `/tools`, `/clear`, `/focus`, `/model`, `/resume`, etc.) using a central alias registry from `hermes_cli/commands.py`.
- Manages the full agent lifecycle: model selection, toolset configuration, session persistence (SQLite), session resumption (`/resume <id>`), checkpointing, and display modes.

---

## Key Structural Details

### HermesCLI `__init__()` (line 4203)
- Parameters: `model`, `toolsets`, `provider`, `reasoning`, `api_key`, `base_url`, `max_turns`, `verbose`, `compact`, `resume`, `checkpoints`, `pass_session_id`, `ignore_rules`.
- Config-driven display features:
  - `tool_progress_mode` — "off", "new", "all", "verbose" (configurable in config.yaml's display section; YAML 1.1 `off` → boolean False is normalised back to string)
  - `focus_view` — reduced-output mode that suppresses per-tool lines (purely cosmetic, does not affect model input)
  - `bell_on_complete`, `show_reasoning`, `reasoning_full`
  - `persistent_output` — keeps output scrollback in memory
- **Input area enhancements:** installs custom key bindings (`shift+enter`, `ctrl+enter`, `cmd+backspace`) and ignores spurious terminal sequences — all in a guarded `try/except` block.
- Uses `prompt_toolkit`'s `Application`, `Layout`, `HSplit`, `Window`, `TextArea`, `FileHistory`, and `KeyBindings` for the TUI.

### `process_command()` (line 9787)
- Normalises the command (lowercase for dispatch, preserves case for args).
- Resolves aliases via `hermes_cli.commands.resolve_command()` — adding an alias is a one-line change.
- Handles `/resume` disarm logic — any non-resume/sessions command clears the pending resume state.
- Commands handled: `quit/exit` (with `--delete` flag), `help`, `profile`, `tools`, `toolsets`, `config`, `redraw`, `clear` (with session discard confirmation), and more.
- `/clear` uses `prompt_toolkit`'s low-level output for screen clearing (Rich's `console.clear()` is swallowed by `patch_stdout`).
- After `/clear`, rebuilds the welcome banner using tool definitions, cwd, model info, and context length.

### Helper utilities at module top
- `format_duration_compact()` — human-readable durations (`45s`, `3m`, `2h 15m`, `1.5d`)
- `format_token_count()` — human-readable token counts (`1.2B`, `456K`, `12.3M`)
- `_reverse_alias_for_display()` — resolves model names through config.yaml's `model_aliases` and `model.aliases` maps for friendly display.
- `CanonicalUsage`, `estimate_usage_cost`, `is_table_divider`, `looks_like_table_row` — all thin wrappers around `agent.usage_pricing` / `agent.markdown_tables`.

### `_STEADY_CURSOR` (line 74)
- Attempts to set a non-blinking block cursor via `prompt_toolkit.cursor_shapes.CursorShape.BLOCK`, gracefully degraded if the import fails.

---

## Notable Constraints / Pitfalls

1. **18,394 lines** — extremely large file. The class is split across three mixins but the file is monolithic. Much of the display logic and TUI rendering lives here directly.
2. **`HERMES_QUIET=1` is set at line 51** — suppresses startup messages from other Hermes modules. This is a side effect of importing `cli.py`.
3. **`prompt_toolkit` TUI integration is delicate** — `redraw` command exists explicitly for recovering from buffer drift caused by multiplexer tab switches, SSH restores, etc. (issue #8688).
4. **Keyboard alias setup is in a bare `try/except`** — if the import of `hermes_cli.pt_input_extras` fails, the key bindings are simply not installed. No fallback or error logged.
5. **Focus view** is a display-only toggle that snaps `tool_progress_mode` to "off" — purely cosmetic, does not affect what the model sees or does.
6. **YAML 1.1 `off` → boolean False** needs explicit normalisation back to the string `"off"` — a common YAML footgun.

---

## Flags / Questions for Human

- [ ] 18,394 lines is a maintenance concern — consider splitting the TUI rendering into a separate module.
- [ ] `HERMES_QUIET=1` at import time is a surprising side effect for anyone importing `cli.py` programmatically.
- [ ] The `_focus_saved_tool_progress` / `_focus_hidden_lines` / `_focus_last_counted_tool` state is updated inline — is the focus view logic tested?
- [ ] Keyboard alias failure is silently swallowed — could help to log a debug message.
- [ ] The session resumption flow (`/resume`) has disarm logic that could be surprising: any non-resume command clears the pending list.

---

## Related Files

- `hermes_cli/cli_agent_setup_mixin.py` — agent initialization mixin
- `hermes_cli/cli_commands_mixin.py` — slash command implementations mixin
- `hermes_cli/cli_billing_mixin.py` — billing display mixin
- `hermes_cli/commands.py` — central command alias registry
- `hermes_cli/fallback_config.py` — `get_fallback_chain`
- `hermes_cli/focus_view.py` — focus mode logic
- `model_tools.py` — for `get_tool_definitions()`
- `run_agent.py` — for `AIAgent` instantiation