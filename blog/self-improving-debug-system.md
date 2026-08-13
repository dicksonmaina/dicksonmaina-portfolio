# Building a Self-Improving Debug System for AI Coding Agents

## The Problem

AI coding agents are powerful, but they keep making the same mistakes. Every session, they re-solve bugs that were already fixed in previous sessions. There's no memory, no learning, no improvement over time.

## The Solution: kilo-auto-debug

I built a **self-improving debugging memory system** that gives AI agents persistent memory, fast lookup, and circuit-breaker safety.

### Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────┐
│  Kilo Agent     │────▶│  debug-mcp   │────▶│   Ollama    │
│  (VS Code/CLI)  │     │  HTTP MCP    │     │  qwen2.5:3b │
└─────────────────┘     └──────┬───────┘     └─────────────┘
                               │
                               ▼
                        ┌──────────────┐
                        │    Redis     │
                        │  kilo:debug  │
                        │  (AOF saved) │
                        └──────┬───────┘
                               │
                        sync.js │ learner.js
                               ▼
                        ┌──────────────┐
                        │ debug_log.jsonl │
                        │  (durable)     │
                        └────────────────┘
```

### Key Components

1. **Redis-backed memory** — every bug, fix, and pattern stored under `kilo:debug:*`
2. **Lightweight MCP server** — HTTP MCP exposing `debug_lookup`, `debug_fix`, `debug_log`
3. **Local Ollama model** — `qwen2.5:3b` for fast AI suggestions (~5-10s)
4. **Circuit breaker** — max 3 auto-debug calls per session
5. **Git safety** — auto-commit before fixes, auto-revert on test failure
6. **Background learner** — analyzes patterns every 60s and improves agent rules

### Why This Works on Low-End Hardware

- **No GPU required**: `qwen2.5:3b` runs on CPU at ~2GB RAM
- **Fast responses**: 5-10s for AI suggestions vs 2+ minutes for cloud agents
- **No cloud dependency**: everything runs locally except optional Kilo Gateway
- **Crash-resistant**: circuit breakers, pre-flight checks, auto-revert

### The Learning Loop

Every debugging cycle logs to Redis:
```
kilo:debug:bug:{id}
kilo:debug:fix:{id}
kilo:debug:pattern:{type}
```

Future sessions query this memory before re-solving known issues. The system gets faster and more accurate over time.

### Safety Guarantees

- All fixes require `git commit` before application
- Failed fixes auto-revert via `git revert HEAD --no-edit`
- Critical services (`hermes-gateway`, `rag-enterprise`, `ubuntu-gateway`) are never auto-fixed
- Circuit breaker prevents infinite debug loops

## What's Next

- GitHub Marketplace listing for the debug MCP server
- Multi-language SDKs (Python, Rust, TypeScript)
- Community plugin system for custom debug patterns
- Integration with more agent frameworks (OpenCode, Claude Code, Codex CLI)

## Try It

```bash
git clone https://github.com/dicksonmaina/kilo-auto-debug ~/.kilo/debug-system
node ~/.kilo/debug-system/.kilo/debug-mcp.js &
```

Full setup guide: https://github.com/dicksonmaina/kilo-auto-debug

---

*Part of the autonomous coding foundation. Built by [dicksonmaina](https://github.com/dicksonmaina).*
