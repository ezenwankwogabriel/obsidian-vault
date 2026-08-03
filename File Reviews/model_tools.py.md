---
type: file-review
file: model_tools.py
module: hermes-agent (tool orchestration)
status: needs-review
reviewed_on: 2026-08-03
tags: [hermes-agent, tools, orchestration, registry]
---

# File Review: `model_tools.py`

**Location:** `/usr/local/lib/hermes-agent/model_tools.py`
**Total lines:** 1,448
**Role:** Thin orchestration layer over the tool registry. Triggers tool discovery, assembles tool definitions for the model, dispatches function calls, and manages toolsets.

---

## What It Does

- **Entry point** for tool discovery: calls `tools.registry.discover_builtin_tools()` which scans `tools/*.py`, imports self-registering modules, and populates the global `registry`.
- **`get_tool_definitions(enabled_toolsets, disabled_toolsets, quiet_mode)`** — the main function that assembles the list of tool schemas sent to the LLM. Resolves composite toolsets (e.g. `hermes-cli`), respects `enabled_toolsets` (intersection) and `disabled_toolsets` (subtraction), handles legacy toolset names, and preserves core tools when platform bundles are disabled (issue #33924).
- **`handle_function_call(function_name, function_args, ...)`** — the dispatcher that routes an LLM-requested tool call to the registry. Has middleware hooks (pre/post tool call), Tool Search bridge dispatch (for `tool_search`/`tool_describe`/`tool_call`), argument coercion via `coerce_tool_args()`, and call logging.
- Provides public API consumed by `run_agent.py`, `cli.py`, `batch_runner.py`, and RL environments.

---

## Key Structural Details

### Async Bridging (lines 51–150)
- **Single source of truth** for sync→async bridging in tool handlers.
- Maintains a persistent main-thread event loop (`_tool_loop`) and per-worker-thread loops (`_worker_thread_local`) to avoid the "Event loop is closed" errors caused by `asyncio.run()` (which destroys the loop after each call, breaking cached httpx/AsyncOpenAI clients).
- `_run_async(coro)` detects whether a loop is already running (gateway async stack, Atropos) and forks a fresh thread with its own loop if needed.
- Uses `concurrent.futures.ThreadPoolExecutor` with explicit task cancellation support.

### `get_tool_definitions()` (around line 300+)
- Starts with built-in core tools (`_HERMES_CORE_TOOLS`) and the MCP tool definition.
- Scopes by `enabled_toolsets` (if provided) via `resolve_toolset()`.
- Applies `disabled_toolsets` as a strict subtraction — with a special guard: platform bundles (`hermes-*`) and posture toolsets only subtract their **non-core** tools so core functionality isn't accidentally stripped (#33924).
- Logs a one-time advisory for disabled platform bundles.
- Caches the resolved tool name list in the process-global `_last_resolved_tool_names`.

### `handle_function_call()` (line 1084 — 220+ lines)
- Signature has 18 parameters: `function_name`, `function_args`, `task_id`, `tool_call_id`, `session_id`, `turn_id`, `api_request_id`, `user_task`, `enabled_tools`, `skip_pre_tool_call_hook`, `skip_tool_request_middleware`, `skip_tool_execution_middleware`, `tool_request_middleware_trace`, `enabled_toolsets`, `disabled_toolsets`.
- **Tool Search bridge** — intercepts `tool_search`, `tool_describe`, `tool_call` bridge names and dispatches them inline; unwraps `tool_call` to the underlying real tool so hooks/guardrails see the real name.
- **Defense in depth:** after resolving an underlying tool via the bridge, it checks that the tool is in the session's scoped deferrable catalog, rejecting out-of-scope calls.
- Calls `registry.dispatch()` for the actual execution, runs middleware, logs results, and handles errors.

---

## Notable Constraints / Pitfalls

1. **The tool registry is global** — all tools self-register into a single `registry` instance when their modules are imported. Thread safety relies on import-time registration being single-threaded.
2. **`disabled_toolsets` subtraction logic is nuanced** — platform bundles like `hermes-desktop` only strip non-core tools. Using a bundle name in `disabled_toolsets` when you meant `toolsets:` is a common mistake and triggers a specific warning (#33924).
3. **`_run_async` is complex** — three code paths (no running loop → shared persistent loop; running loop → disposable worker thread; worker thread → per-thread persistent loop). This is a frequent source of subtle bugs.
4. **`coerce_tool_args()`** runs at the top of `handle_function_call()` — attempts to cast string arguments to their schema-declared types (e.g. `"42"` → `42`). Can mask mismatches silently.
5. **The Tool Search bridge scope** depends on `enabled_toolsets`/`disabled_toolsets` being correctly threaded through — a mismatch would let a restricted session see or call tools it shouldn't.

---

## Flags / Questions for Human

- [ ] The `_run_async` branching logic is critical and complex — worth testing under gateway, CLI, and worker-thread scenarios.
- [ ] How often is `tool_request_middleware_trace` populated? Heavy middleware could slow dispatch.
- [ ] The `_WARNED_DISABLED_BUNDLES` set is never cleared — safe since it's process-lifetime.
- [ ] Tool discovery cache (`tool_discovery_cache.json`) — verify the mtime+size heuristic works reliably under version control.

---

## Related Files

- `tools/registry.py` — `discover_builtin_tools`, `registry.dispatch()`, `tool_error()`
- `toolsets/__init__.py` — `resolve_toolset()`, `validate_toolset()`, `bundle_non_core_tools()`
- `run_agent.py` — imports and uses all `model_tools` public API
- `cli.py` — calls `get_tool_definitions()` at startup and `handle_function_call()` during agent turns