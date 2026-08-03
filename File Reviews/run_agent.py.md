---
type: file-review
file: run_agent.py
module: hermes-agent (core runner)
status: needs-review
reviewed_on: 2026-08-03
tags: [hermes-agent, aiagent, core-loop]
---

# File Review: `run_agent.py`

**Location:** `/usr/local/lib/hermes-agent/run_agent.py`
**Total lines:** 7,551
**Role:** The central AIAgent class — the agent "runner" that owns the conversation loop, tool-calling lifecycle, and model provider interface.

---

## What It Does

- Defines the `AIAgent` class, which is the entry point for running a Hermes agent session.
- `__init__` accepts ~80 parameters (model, provider, callbacks, session context, checkpoint config, credential pool, etc.) but delegates **all** initialization to `agent.agent_init.init_agent()` — the constructor is a pure forwarder.
- `run_conversation()` is also a forwarder to `agent.conversation_loop.run_conversation()`, wrapped in relay-runtime lease acquisition, portal tagging, accounting context, and subagent lifecycle binding.
- Handles bootstrap (pre-import of `hermes_bootstrap` for Windows UTF-8 support), lazy OpenAI SDK import (proxy object to save ~240ms), .env loading via `load_hermes_dotenv()`, and tool module imports from `model_tools`.

---

## Key Structural Details

### AIAgent Class (`line 409`)
- **Property:** `base_url` with a setter that also caches `.lower()` and the hostname for fast path matching.
- **Constructor** is a thin forwarder to `init_agent()` in `agent/agent_init.py` (lines 432–589).
- Has its own `_TOOL_CALL_ARGUMENTS_CORRUPTION_MARKER` constant for graceful degradation on corrupted tool call data (issue #15236).

### Session-awareness
- `_launch_cwd_for_session(source)` — only records cwd for local CLI sessions; skips for gateway/cron/remote-backend.
- `_session_source_for_agent(platform)` — resolves session source from gateway context or env var, falling back to the platform string or `"cli"`.
- Session ID, parent session ID, platform, user metadata all passed in and threaded through to accounting/relay.

### Conversation Loop (`run_conversation`, `line 7153`)
- Wraps the core `run_conversation()` from `agent/conversation_loop` with:
  - Relay-runtime lease + turn lifecycle (`SESSION_COORDINATOR.acquire_conversation`, `.begin_turn`)
  - Portal / Nous tagging context (`set_conversation_context`)
  - Accounting context setup (`set_accounting_context`) for auxiliary LLM call tracking (fixes #23270)
  - Subagent parent binding (`bind_subagent_parent`)
  - Scoped runtime main context
- Returns a `Dict[str, Any]` with the final result.

### Test compatibility
- Re-exports `OpenAI`, `_SafeWriter`, `get_tool_definitions`, `handle_function_call`, `check_toolset_requirements`, `estimate_request_tokens_rough`, `ContextCompressor`, prompt builders, sanitization utils, and more — all annotated `# noqa: F401` for `mock.patch("run_agent.<name>", ...)` patterns used by ~28 test files.

---

## Notable Constraints / Pitfalls

1. **No direct `from openai import OpenAI` at module top** — lazy proxy via `agent.process_bootstrap` to keep import time low. Tests must patch `run_agent.OpenAI` (not `openai.OpenAI`).
2. **`fire` is imported only in `__main__`** — avoids breaking daemon threads on partial installs.
3. **The `tool_delay` parameter is deprecated silently** — accepted for backward compat but ignored; sequential tool calls no longer sleep.
4. **`.env` file loading is eager at import time** — happens before `if __name__ == "__main__"` even runs.
5. **7,551 lines is large** — the class is a thin shell that delegates; actual logic lives in `agent/` subpackage.
6. **Error resilience is layered** — bootstrap failure is caught silently, corrupt tool call arguments produce a constant marker instead of crashing.

---

## Flags / Questions for Human

- [ ] 7,551 lines for what is essentially a forwarder facade — could there be dead or vestigial parameters in `__init__`?
- [ ] The `.env` loading at module import time side-effects globally — intentional for a CLI tool but worth noting.
- [ ] Many re-exports exist purely for test patching — are any still used by real callers?

---

## Related Files

- `agent/agent_init.py` — where `AIAgent.__init__` actually does its work
- `agent/conversation_loop.py` — where `run_conversation` actually lives
- `model_tools.py` — tool orchestration (imported by this file)
- `tools/registry.py` — `discover_builtin_tools` for tool registration
- `hermes_bootstrap.py` — UTF-8 stdio bootstrap