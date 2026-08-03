---
type: moc
tags: [index, file-review]
---

# File Reviews — Index

Notes the agent has mapped for later review. One note per file under `File Reviews/`.

## Dataview (如果你的客户端装了 Dataview 插件)
```dataview
TABLE module, status, reviewed_on FROM "File Reviews"
WHERE type = "file-review"
SORT status DESC, file.name ASC
```

## Manual list
- [[pyproject.toml]] — build/deps manifest (version cap, exact pins)
- [[AGENTS.md]] — dev guide, contribution rubric, footprint ladder
- [[file-review skill]] — the agent skill that automates this whole workflow
- [[run_agent.py]] — AIAgent class, conversation loop, __init__ params (7,551 lines, forwarder to agent/ subpackage)
- [[model_tools.py]] — tool orchestration: discovery, tool definitions assembly, handle_function_call dispatch (1,448 lines)
- [[cli.py]] — interactive REPL/TUI, HermesCLI class, slash commands (18,394 lines, mixin-based)
- [[Project Overview]] — setup summary, architecture, conventions, what was parked

## Status legend
- `needs-review` — mapped, not yet verified by a human
- `reviewed` — you've eyeballed it
- `flagged` — something looks off, see the note's flags section
