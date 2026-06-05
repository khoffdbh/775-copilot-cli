# Context Mode Examples

## 1) Debug an API endpoint (ctx_execute, JavaScript)
```javascript
// ctx_execute: fetch and summarize orders
const resp = await fetch('http://localhost:3000/api/orders');
const { orders } = await resp.json();
const negQty = orders.filter(o => o.quantity < 0);
console.log(`${orders.length} orders; negative qty: ${negQty.length}`);
if (negQty.length) console.log('IDs:', negQty.map(o => o.id).join(', '));
```

## 2) Read and analyze a large JSON file (ctx_execute_file, Python)
```python
# FILE_CONTENT provided by ctx_execute_file
import json
data = json.loads(FILE_CONTENT)
print('records:', len(data))
# simple distribution example
from collections import Counter
print('types:', Counter(item.get('type') for item in data).most_common(10))
```

## 3) Fetch and index external docs (ctx_fetch_and_index)
```text
# Single call: index React docs for later search
ctx_fetch_and_index(requests: [{url: 'https://react.dev/reference/react', source: 'react.dev'}], concurrency: 2)
# then
ctx_search(queries: ['useEffect cleanup pattern', 'useState vs useReducer'], source: 'react.dev')
```

## 4) Playwright snapshot workflow (filename + ctx_index)
```text
browser_snapshot(filename: '/tmp/playwright-snapshot.md')
ctx_index(path: '/tmp/playwright-snapshot.md', source: 'playwright-snapshot')
ctx_search(queries: ['login form email password'], source: 'playwright-snapshot')
```

## 5) Hook script minimal pattern (run-hook.mjs)
```mjs
import { runHook } from '../../hooks/run-hook.mjs';
await runHook(async () => {
  const stdin = await (await import('../../hooks/core/stdin.mjs')).readStdin();
  const input = (await import('../../hooks/session-helpers.mjs')).parseStdin(stdin);
  // route or implement logic here
  process.stdout.write(JSON.stringify({ action: 'allow' }) + '\n');
});
```

Notes
- Batch ctx_search queries into a single call (queries array).
- Prefer ctx_index(path) over ctx_index(content) for large data.
- Always use `filename` parameters for Playwright exports and browser console/network captures.
