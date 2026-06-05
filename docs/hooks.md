# Hooks (overview)

Purpose
- Hooks run at lifecycle points (PreToolUse, PostToolUse, sessionstart, stop, etc.) to route, format, or instrument tool usage. They are the integration layer that enforces context-mode routing.

Important runtime pieces
- hooks/run-hook.mjs: provides runHook(handler) — a crash-resilient wrapper that dynamically imports helper side-effects, logs failures to <configDir>/context-mode/hook-errors.log, and guarantees a zero exit code so hook failures do not spam users.
- hooks/pretooluse.mjs: example PreToolUse hook that routes CLI/tool requests into context-mode (calls routePreToolUse), writes markers for downstream PostToolUse, and ensures cross-platform behavior.
- src/util/hook-config.ts: helpers to extract commands from hook entries, parse node command strings, and compute hook script paths for adapters/plugins.

Config
- adapters and plugins declare their hooks via hooks.json and/or adapter.generateHookConfig(pluginRoot).
- configs/*/hooks.json (and per-adapter generateHookConfig) list hook entries. Use getCommandsFromHookEntry() and extractHookScriptPath() helpers to enumerate script paths.

Testing and safety
- Many tests exist under tests/hooks/* (latency, integration, platform normalization). Use them as examples of expected hook shapes and edge cases (Windows path quoting, legacy command shapes, and migration behaviors).

Guidance for doc authors
- Explain runHook contract: never throw out-of-band, log failures, and exit 0 on errors.
- Document PreToolUse routing behavior: when tools are considered "large-output" and are redirected to ctx_execute/ctx_execute_file/ctx_fetch_and_index.
- Show config examples (hooks.json entries) and how to author a hook script that uses runHook:

```mjs
import { runHook } from "./run-hook.mjs";
await runHook(async () => {
  // parse stdin
  // route via core/routing.mjs or implement custom logic
  // write JSON to stdout
});
```

- Provide migration notes: how legacy quoted/unquoted node commands are parsed, and why scripts should be emitted as fully-quoted node commands (use buildNodeCommand/parseNodeCommand helpers).

Next steps
- Populate docs/examples/context-mode-examples.md with runnable snippets taken from tests/hooks and skills/context-mode guidance.
- Link these docs from the site TOC and add a CHANGELOG entry.

