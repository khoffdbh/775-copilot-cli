# Context Mode (MCP)

Purpose
- Keep large outputs out of the assistant context by using context-mode MCP tools.
- Use ctx_execute / ctx_execute_file / ctx_fetch_and_index / ctx_index / ctx_search for analysis, indexing and searching.

Mandatory rule
- Default to context-mode for any command that may produce large output. Only run small, safe commands via Bash (file mutations, git writes, navigation, process control, package installs, and simple echo).

When to use each tool
- ctx_execute: Run commands or fetch APIs where output needs processing in a sandbox (JavaScript, Python, shell). Example: `ctx_execute` to run `gh pr list` and print a short summary.
- ctx_execute_file: Read large files (logs, JSON, CSV) into FILE_CONTENT for sandboxed analysis.
- ctx_fetch_and_index: Fetch external HTML/docs and index them server-side for later ctx_search queries.
- ctx_index(path): Index local files server-side (prefer path over content for large data).
- ctx_search(queries): Search indexed content (batch all queries in one call).
- ctx_stats / ctx_doctor / ctx_upgrade / ctx_purge: maintenance and diagnostics.

Key rules and anti-patterns
- Always use `filename` when Playwright snapshots or console/network exports are available; then process via ctx_index(path) or ctx_execute_file(path).
- NEVER call `ctx_index(content: largeData)` — sending large data via the content parameter floods the context.
- Do not re-index MCP tool outputs that are already in context; either use them directly or save to file and index.
- stdout is the only thing that enters context from ctx_execute/ctx_execute_file — print concise, specific findings.

Playwright & browser workflow (example)
1. browser_snapshot(filename: "/tmp/snap.md") → saves to file (tiny confirmation)
2. ctx_index(path: "/tmp/snap.md", source: "Playwright snapshot")
3. ctx_search(queries: ["login form email password"], source: "Playwright snapshot")

Example: Analyze an API endpoint (JavaScript)
```javascript
const resp = await fetch('http://localhost:3000/api/orders');
const { orders } = await resp.json();
const negQty = orders.filter(o => o.quantity < 0);
console.log(`${orders.length} orders; negative qty: ${negQty.length}`);
```

Best practices
- Batch related ctx_search queries in a single call (use the queries array).
- Prefer server-side path-based indexing; pass small inline content only for handcrafted short text.
- When uncertain, use context-mode rather than ad-hoc Bash that might leak large data.

References
- docs/examples/context-mode-examples.md (runnable snippets)
- docs/hooks.md (how hooks route tools into context-mode)

