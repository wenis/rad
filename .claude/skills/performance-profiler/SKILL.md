---
name: performance-profiler
description: Profiles application performance to identify bottlenecks, slow queries, memory leaks, and optimization opportunities. Use when investigating slow performance OR before optimization OR analyzing production issues OR improving response times.
allowed-tools: Read, Grep, Bash, Write
---

# Performance Profiler

You profile applications to identify performance bottlenecks and provide specific optimization recommendations.

## When to use
- Investigating slow API endpoints
- Before deploying to production (performance baseline)
- After user complaints about slowness
- Analyzing database query performance
- Finding memory leaks
- Optimizing resource usage
- Preparing for load testing

## Performance Categories

### 1. CPU Usage
- Hot code paths consuming CPU
- Inefficient algorithms
- Unnecessary computations

### 2. Database Performance
- Slow queries (N+1 problems)
- Missing indexes
- Full table scans
- Connection pool issues

### 3. Memory Usage
- Memory leaks
- Large object allocations
- Inefficient data structures

### 4. I/O Performance
- File system operations
- Network requests
- External API calls

### 5. Concurrency Issues
- Thread contention
- Lock waiting
- Blocking operations

## Profiling Tools

### Python

**cProfile (built-in):**
```bash
# Profile a script
python -m cProfile -s cumulative script.py

# Save profile data
python -m cProfile -o profile.stats script.py

# Analyze with snakeviz
pip install snakeviz
snakeviz profile.stats
```

**py-spy (production-safe):**
```bash
# Install
pip install py-spy

# Profile running process
py-spy top --pid 12345

# Generate flame graph
py-spy record -o profile.svg --pid 12345

# Profile for 60 seconds
py-spy record -o profile.svg --duration 60 --pid 12345
```

**memory_profiler:**
```bash
# Install
pip install memory_profiler

# Decorate function
from memory_profiler import profile

@profile
def my_func():
    big_list = [1] * (10 ** 6)
    return big_list

# Run
python -m memory_profiler script.py
```

### JavaScript/Node.js

**Built-in profiler:**
```bash
# Profile with V8
node --prof app.js

# Process profile
node --prof-process isolate-0xnnnnnnnnnnnn-v8.log > processed.txt
```

**clinic.js:**
```bash
# Install
npm install -g clinic

# Profile CPU (flame graph)
clinic flame -- node app.js

# Profile event loop
clinic doctor -- node app.js

# Profile memory
clinic heapprofiler -- node app.js
```

**Chrome DevTools:**
```javascript
// In code: trigger profiler
const inspector = require('inspector');
const fs = require('fs');
const session = new inspector.Session();
session.connect();

session.post('Profiler.enable', () => {
  session.post('Profiler.start', () => {
    // Run code to profile
    runExpensiveOperation();

    session.post('Profiler.stop', (err, { profile }) => {
      fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
    });
  });
});
```

### Go

**pprof (built-in):**
```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()

    // Application code
}
```

```bash
# CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Memory profile
go tool pprof http://localhost:6060/debug/pprof/heap

# Goroutine profile
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

## Database Profiling

### PostgreSQL

**Find slow queries:**
```sql
-- Enable query logging
ALTER SYSTEM SET log_min_duration_statement = 1000; -- Log queries > 1s
SELECT pg_reload_conf();

-- View slow queries
SELECT
  query,
  calls,
  total_time,
  mean_time,
  max_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Explain query
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE email = 'test@example.com';
```

**Find missing indexes:**
```sql
-- Tables with sequential scans
SELECT
  schemaname,
  tablename,
  seq_scan,
  seq_tup_read,
  idx_scan,
  seq_tup_read / seq_scan AS avg_seq_read
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC
LIMIT 10;

-- Unused indexes
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexname NOT LIKE 'pg_toast%'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### MySQL

**Enable slow query log:**
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
```

**Analyze slow queries:**
```bash
# Use pt-query-digest
pt-query-digest /var/log/mysql/slow.log

# Or mysqldumpslow
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log
```

**Explain query:**
```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

### MongoDB

**Profile queries:**
```javascript
// Enable profiling
db.setProfilingLevel(2); // Profile all queries

// View slow queries
db.system.profile.find({millis: {$gt: 100}}).sort({millis: -1}).limit(10);

// Explain query
db.users.find({email: 'test@example.com'}).explain('executionStats');
```

## Profiling Patterns

### Profile API Endpoint (Python/Flask)

**Add profiling middleware:**
```python
from werkzeug.middleware.profiler import ProfilerMiddleware

app = Flask(__name__)
app.wsgi_app = ProfilerMiddleware(app.wsgi_app, restrictions=[30], profile_dir='./profiles')
```

**Or use custom decorator:**
```python
import cProfile
import pstats
from functools import wraps

def profile(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()

        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(20)

        return result
    return wrapper

@app.route('/slow-endpoint')
@profile
def slow_endpoint():
    # Endpoint code
    return jsonify({"status": "ok"})
```

### Profile API Endpoint (Node.js/Express)

**Add timing middleware:**
```javascript
const app = express();

app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    if (duration > 1000) {
      console.log(`Slow request: ${req.method} ${req.url} - ${duration}ms`);
    }
  });

  next();
});
```

**Profile specific endpoint:**
```javascript
const { performance } = require('perf_hooks');

app.get('/users', async (req, res) => {
  const start = performance.now();

  const dbStart = performance.now();
  const users = await db.query('SELECT * FROM users');
  const dbTime = performance.now() - dbStart;

  const serializationStart = performance.now();
  const json = JSON.stringify(users);
  const serializationTime = performance.now() - serializationStart;

  const total = performance.now() - start;

  console.log({
    total: `${total.toFixed(2)}ms`,
    db: `${dbTime.toFixed(2)}ms`,
    serialization: `${serializationTime.toFixed(2)}ms`
  });

  res.json(users);
});
```

### Find N+1 Query Problems

**Problem example:**
```python
# BAD: N+1 queries
users = User.query.all()  # 1 query
for user in users:
    print(user.posts)  # N queries (one per user!)
```

**Solution:**
```python
# GOOD: 1 query with join
users = User.query.options(joinedload(User.posts)).all()
for user in users:
    print(user.posts)  # No additional queries
```

**Detection:**
```python
# Django: Enable query logging
import logging
logging.basicConfig()
logging.getLogger('django.db.backends').setLevel(logging.DEBUG)

# SQLAlchemy: Echo queries
engine = create_engine('postgresql://...', echo=True)
```

### Profile Memory Usage

**Python:**
```python
from memory_profiler import profile

@profile
def load_large_dataset():
    data = []
    for i in range(1000000):
        data.append({'id': i, 'value': i * 2})
    return data

# Run
python -m memory_profiler script.py
```

**Output:**
```
Line #    Mem usage    Increment   Line Contents
================================================
     3   38.6 MiB     38.6 MiB   @profile
     4                             def load_large_dataset():
     5   38.6 MiB      0.0 MiB       data = []
     6  344.3 MiB    305.7 MiB       for i in range(1000000):
     7  344.3 MiB      0.0 MiB           data.append({'id': i, 'value': i * 2})
     8  344.3 MiB      0.0 MiB       return data
```

**Node.js:**
```javascript
const { heapUsed } = process.memoryUsage();
console.log(`Heap used: ${(heapUsed / 1024 / 1024).toFixed(2)} MB`);

// Check for memory leaks
if (global.gc) {
  global.gc();
  const used = process.memoryUsage().heapUsed / 1024 / 1024;
  console.log(`After GC: ${used.toFixed(2)} MB`);
}
```

## Performance Report Format

```markdown
# Performance Analysis Report

## Summary
- **Average Response Time**: 450ms (target: < 200ms)
- **P95 Response Time**: 1,200ms
- **P99 Response Time**: 2,500ms
- **Database Time**: 65% of total time
- **CPU Usage**: 45% average, 85% peak
- **Memory Usage**: 2.1GB / 4GB (53%)

## Critical Bottlenecks

### 1. Slow Database Query in User List Endpoint
**Endpoint**: `GET /api/users`
**Issue**: N+1 query problem loading user posts
**Impact**: 1,200ms average response time (should be < 200ms)

**Current Code:**
```python
users = User.query.all()  # 1 query
for user in users:
    posts_count = len(user.posts)  # N queries!
```

**Query Analysis:**
- Base query: 15ms
- N additional queries: 1,185ms (100 users × ~12ms each)

**Recommendation:**
```python
# Use eager loading
users = User.query.options(
    selectinload(User.posts)
).all()
```

**Expected Improvement**: 1,200ms → 50ms (24x faster)

---

### 2. Memory Leak in Cache Service
**Service**: `CacheService`
**Issue**: Cache entries never evicted, growing unbounded
**Impact**: Memory grows from 500MB → 3GB over 24 hours

**Problem Code:**
```javascript
class CacheService {
  constructor() {
    this.cache = new Map();
  }

  set(key, value) {
    this.cache.set(key, value);  // Never deleted!
  }
}
```

**Recommendation:**
```javascript
// Use LRU cache with size limit
const LRU = require('lru-cache');

class CacheService {
  constructor() {
    this.cache = new LRU({
      max: 1000,  // Max 1000 items
      maxAge: 1000 * 60 * 60  // 1 hour TTL
    });
  }

  set(key, value) {
    this.cache.set(key, value);
  }
}
```

**Expected Improvement**: Memory stable at ~800MB

---

### 3. Missing Database Index
**Table**: `orders`
**Column**: `created_at`
**Issue**: Full table scan on date range queries

**Query:**
```sql
SELECT * FROM orders WHERE created_at > '2025-01-01';
```

**Execution Plan:**
```
Seq Scan on orders  (cost=0.00..15234.00 rows=50000 width=200)
  Filter: (created_at > '2025-01-01'::date)
```

**Recommendation:**
```sql
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

**Expected Improvement**: 850ms → 12ms (70x faster)

## Performance by Endpoint

| Endpoint | Requests | Avg Time | P95 Time | Issues |
|----------|----------|----------|----------|--------|
| GET /api/users | 12,450 | 1,200ms | 2,100ms | N+1 queries |
| POST /api/orders | 3,200 | 450ms | 890ms | Missing index |
| GET /api/products | 45,600 | 85ms | 120ms | ✅ Good |

## Database Query Analysis

### Slowest Queries

1. **User list with posts**
   - Query time: 1,185ms
   - Calls: 100 per request
   - Fix: Eager loading

2. **Order date range**
   - Query time: 850ms
   - Calls: 1 per request
   - Fix: Add index on `created_at`

3. **Product search**
   - Query time: 420ms
   - Calls: 1 per request
   - Fix: Full-text search index

## Recommendations by Priority

### Critical (Fix Immediately)
1. Add eager loading for user posts (24x improvement)
2. Add index on orders.created_at (70x improvement)
3. Fix memory leak in CacheService

### High (Fix This Week)
1. Add full-text search index for products
2. Implement connection pooling (currently creating new connections)
3. Add caching for frequently accessed data

### Medium (Plan for Next Sprint)
1. Optimize image processing pipeline
2. Implement pagination for large result sets
3. Add database read replicas

## Optimization Checklist

### Immediate Actions
- [ ] Add database indexes (orders.created_at, products.name)
- [ ] Fix N+1 query problems (user posts, order items)
- [ ] Implement LRU cache with TTL
- [ ] Enable query caching in ORM

### Short-term Actions
- [ ] Add APM tool (New Relic, DataDog, Elastic APM)
- [ ] Set up performance budgets in CI/CD
- [ ] Profile production traffic for 1 week
- [ ] Implement database connection pooling

### Long-term Actions
- [ ] Migrate hot data to faster storage
- [ ] Implement horizontal scaling
- [ ] Add CDN for static assets
- [ ] Optimize database schema design
```

## Instructions

1. **Identify performance issues:**
   - User reports slow endpoints?
   - Check application metrics
   - Review database logs
   - Profile CPU/memory usage

2. **Profile the application:**
   - Use appropriate profiling tool
   - Focus on hot code paths
   - Measure database query times
   - Check memory allocations

3. **Analyze bottlenecks:**
   - Rank by impact (frequency × duration)
   - Identify root causes
   - Estimate improvement potential

4. **Create recommendations:**
   - Specific code changes
   - Expected performance gains
   - Implementation difficulty
   - Testing approach

5. **Write report:**
   - Summary with key metrics
   - Critical bottlenecks with fixes
   - Prioritized action items
   - Before/after comparisons

## Best Practices

✅ **DO:**
- Profile in production-like environment
- Measure before and after changes
- Focus on biggest bottlenecks first
- Use APM tools for continuous monitoring
- Set performance budgets

❌ **DON'T:**
- Optimize prematurely
- Profile in development only
- Make changes without measuring impact
- Ignore database performance
- Optimize without profiling data

## Constraints

- Profile with realistic data volumes
- Don't profile in production without safety measures
- Measure impact before claiming improvement
- Test optimizations thoroughly
- Document all performance changes
