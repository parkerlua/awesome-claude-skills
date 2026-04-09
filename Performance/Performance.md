---
name: performance
description: Identify and fix performance bottlenecks. Use when asked to optimize code, improve load times, reduce memory usage, or profile slow operations.
---

You are optimizing for performance. Measure first, optimize second. Never guess what's slow — find it.

## The Process

1. **Measure** — get a baseline. What is the actual number? (ms, MB, req/s)
2. **Profile** — find where time/memory is actually spent
3. **Identify** — name the specific bottleneck
4. **Fix** — change one thing at a time
5. **Verify** — measure again. Did it actually improve?

Don't optimize code that isn't slow. Don't optimize before you have data.

## Frontend Performance

### Find the bottleneck first

```javascript
// Browser DevTools: Performance tab → Record → Profile
// Look for: Long Tasks (>50ms), layout thrashing, excessive re-renders

// Measure specific operations
console.time('render')
// ... operation
console.timeEnd('render')

// React: use React DevTools Profiler
// Look for: components that re-render when they shouldn't
```

### Unnecessary Re-renders (React)

```tsx
// 🐌 Re-renders every time parent renders
function ExpensiveList({ items, onSelect }) {
  return items.map(item => <Item key={item.id} item={item} onSelect={onSelect} />)
}

// ✅ Only re-renders when items or onSelect actually change
const ExpensiveList = memo(function ExpensiveList({ items, onSelect }) {
  return items.map(item => <Item key={item.id} item={item} onSelect={onSelect} />)
})

// ✅ Stable function reference — doesn't break memo
const handleSelect = useCallback((id) => {
  setSelected(id)
}, []) // empty deps = created once
```

### Expensive Calculations

```tsx
// 🐌 Recalculates on every render
function Dashboard({ orders }) {
  const total = orders.reduce((sum, o) => sum + o.amount, 0)
  const byRegion = groupBy(orders, 'region') // expensive
}

// ✅ Only recalculates when orders changes
function Dashboard({ orders }) {
  const total = useMemo(() => orders.reduce((sum, o) => sum + o.amount, 0), [orders])
  const byRegion = useMemo(() => groupBy(orders, 'region'), [orders])
}
```

### Bundle Size

```bash
# Analyze what's in your bundle
npx vite-bundle-visualizer
npx webpack-bundle-analyzer

# Common culprits:
# - moment.js (use date-fns or dayjs instead)
# - lodash (import specific functions: import debounce from 'lodash/debounce')
# - importing an entire icon library for 3 icons
```

**Code splitting:**
```tsx
// 🐌 All loaded upfront
import { HeavyChart } from './HeavyChart'

// ✅ Loaded only when needed
const HeavyChart = lazy(() => import('./HeavyChart'))
```

## Backend Performance

### Find slow operations

```typescript
// Simple timing wrapper
async function timed<T>(label: string, fn: () => Promise<T>): Promise<T> {
  const start = performance.now()
  const result = await fn()
  console.log(`${label}: ${(performance.now() - start).toFixed(2)}ms`)
  return result
}

// Use APM tools: Datadog, New Relic, OpenTelemetry
// Look for: slow endpoints, high p99 latency, memory growth over time
```

### N+1 Queries

```typescript
// 🐌 1 query + N queries (one per post)
const posts = await Post.findAll()
for (const post of posts) {
  post.author = await User.findByPk(post.userId) // N queries!
}

// ✅ 2 queries total
const posts = await Post.findAll({ include: [{ model: User, as: 'author' }] })

// ✅ Or manual batching
const posts = await Post.findAll()
const userIds = [...new Set(posts.map(p => p.userId))]
const users = await User.findAll({ where: { id: userIds } })
const userMap = Object.fromEntries(users.map(u => [u.id, u]))
posts.forEach(p => p.author = userMap[p.userId])
```

### Caching

```typescript
// Cache expensive, rarely-changing data
const cache = new Map<string, { data: any; expires: number }>()

async function getCachedData(key: string, fetchFn: () => Promise<any>, ttlMs = 60_000) {
  const cached = cache.get(key)
  if (cached && cached.expires > Date.now()) return cached.data

  const data = await fetchFn()
  cache.set(key, { data, expires: Date.now() + ttlMs })
  return data
}

// For production: Redis with appropriate TTLs
// Cache: computed aggregates, external API responses, expensive DB queries
// Don't cache: user-specific data in shared cache, frequently changing data
```

### Async & Concurrency

```typescript
// 🐌 Sequential — waits for each before starting the next
const user = await getUser(id)
const posts = await getPosts(id)
const followers = await getFollowers(id)

// ✅ Parallel — all start at the same time
const [user, posts, followers] = await Promise.all([
  getUser(id),
  getPosts(id),
  getFollowers(id)
])
```

## Memory Performance

```typescript
// Find memory leaks: Node.js
// node --inspect your-app.js
// Chrome DevTools → Memory → Take heap snapshot over time

// Common leak patterns:

// 🚨 Event listener never removed
window.addEventListener('resize', handler) // added on every render
// ✅ Cleanup in useEffect return / component unmount
useEffect(() => {
  window.addEventListener('resize', handler)
  return () => window.removeEventListener('resize', handler)
}, [])

// 🚨 Growing cache with no eviction
const cache = new Map() // grows forever
// ✅ Use LRU cache with size limit
import LRU from 'lru-cache'
const cache = new LRU({ max: 500 })
```

## Output Format

When reporting a performance investigation:

```
## Performance Analysis

**Baseline:** [measured number before any changes]

### Bottleneck Found
[What is slow, where, and by how much]

### Root Cause
[Why it's slow]

### Fix
[The specific change]

### Result
[Measured number after fix — always measure, never estimate]

### Other Opportunities (lower priority)
[Other things noticed that aren't critical]
```
