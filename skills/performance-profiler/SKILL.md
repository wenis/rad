---
name: performance-profiler
description: Profiles application performance using Chrome DevTools, Python cProfile, Node.js --inspect, or Go pprof to identify CPU bottlenecks, memory leaks, slow database queries (N+1 problems), and optimization opportunities. Generates flame graphs, analyzes heap snapshots, identifies hot paths, and provides actionable recommendations. Use when investigating slow API responses, debugging memory leaks, optimizing database queries, analyzing production performance issues, or preparing for scale.
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
- Hot code paths consuming CPU cycles
- Inefficient algorithms (O(n²) where O(n) possible)
- Unnecessary computations in loops

### 2. Database Performance
- Slow queries (N+1 problems)
- Missing indexes
- Full table scans
- Connection pool exhaustion

### 3. Memory Usage
- Memory leaks (objects not garbage collected)
- Large object allocations
- Inefficient data structures

### 4. I/O Performance
- File system operations
- Network requests
- External API calls
- Blocking I/O

### 5. Concurrency Issues
- Thread contention
- Lock waiting
- Deadlocks
- Blocking operations in async code

## Quick Profiling Guide

### Python - cProfile (Built-in)

```bash
# Profile a script
python -m cProfile -s cumulative script.py

# Save and visualize
python -m cProfile -o profile.stats script.py
pip install snakeviz
snakeviz profile.stats  # Opens browser with interactive visualization
```

**Output interpretation:**
- `ncalls`: Number of calls
- `tottime`: Total time in function (excluding subcalls)
- `cumtime`: Cumulative time (including subcalls)
- Focus on high `cumtime` functions

### Node.js - clinic.js

```bash
npm install -g clinic

# CPU profiling (flame graph)
clinic flame -- node app.js

# Event loop issues
clinic doctor -- node app.js

# Memory profiling
clinic heapprofiler -- node app.js
```

### Go - pprof (Built-in)

```go
import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    // Your app code
}
```

```bash
# CPU profile
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Memory profile
go tool pprof http://localhost:6060/debug/pprof/heap

# View in browser
go tool pprof -http=:8080 profile.pb.gz
```

## Complete Tool Guides

For detailed profiling examples and advanced techniques:

**Python Profiling** → `examples/python-profiling.md`
- cProfile deep dive
- py-spy for production
- memory_profiler
- line_profiler
- Profiling async code

**Node.js Profiling** → `examples/nodejs-profiling.md`
- clinic.js toolkit
- Chrome DevTools
- 0x flame graphs
- Heap snapshots
- Production profiling

**Go Profiling** → `examples/go-profiling.md`
- pprof CPU/memory/goroutine profiling
- Flame graphs
- Benchmarking
- Race detection

**Database Profiling** → `reference/database-profiling.md`
- PostgreSQL EXPLAIN ANALYZE
- MySQL slow query log
- N+1 query detection
- Index optimization

**Frontend Performance** → `reference/frontend-profiling.md`
- Chrome DevTools Performance tab
- Lighthouse
- React Profiler
- Bundle analysis

## Common Performance Issues

### N+1 Query Problem

**Problem:**
```python
# Bad: N+1 queries
users = User.query.all()  # 1 query
for user in users:
    posts = user.posts  # N queries (one per user)
```

**Solution:**
```python
# Good: Eager loading
users = User.query.options(joinedload(User.posts)).all()  # 1 query
for user in users:
    posts = user.posts  # No additional queries
```

### Memory Leak

**Detect:**
```bash
# Node.js - Take heap snapshots over time
node --inspect app.js
# Chrome DevTools -> Memory -> Take snapshot
# Compare snapshots to find growing objects
```

**Common causes:**
- Event listeners not removed
- Closures holding references
- Global variables accumulating data
- Cache without eviction policy

### Blocking I/O

**Problem:**
```javascript
// Bad: Blocks event loop
const data = fs.readFileSync('large-file.txt');
```

**Solution:**
```javascript
// Good: Async I/O
const data = await fs.promises.readFile('large-file.txt');
```

### Inefficient Algorithm

**Problem:**
```python
# O(n²) - nested loops
def has_duplicates(items):
    for i in range(len(items)):
        for j in range(i+1, len(items)):
            if items[i] == items[j]:
                return True
    return False
```

**Solution:**
```python
# O(n) - use set
def has_duplicates(items):
    return len(items) != len(set(items))
```

## Profiling Checklist

Before profiling:
- [ ] Reproduce the performance issue
- [ ] Identify slow endpoint/operation
- [ ] Check production metrics (if available)
- [ ] Prepare test data/load

During profiling:
- [ ] Profile in realistic environment
- [ ] Use production-like data volume
- [ ] Profile multiple times (warm vs cold cache)
- [ ] Isolate the bottleneck

After profiling:
- [ ] Identify top 3 bottlenecks
- [ ] Estimate impact of each fix
- [ ] Prioritize highest impact optimizations
- [ ] Document baseline metrics
- [ ] Verify improvements with profiling

## Interpreting Results

### Flame Graphs

**How to read:**
- Width = time spent in function
- Height = call stack depth
- Look for wide plateaus = bottlenecks
- Ignore thin spikes unless very tall

### Database EXPLAIN

```sql
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'test@example.com';
```

**Look for:**
- `Seq Scan` on large tables = missing index
- High `cost` values = expensive operation
- `Actual time` >> `estimated` = outdated statistics

### Memory Profiling

**Key metrics:**
- Heap size growth over time
- Number of objects created
- Retained size (objects + references)
- Garbage collection frequency

**Red flags:**
- Linear memory growth = leak
- High GC time (>10%) = too many allocations
- Large retained heap = inefficient data structures

## Optimization Priorities

### 1. Database Queries (Highest Impact)
- Add indexes for frequent queries
- Fix N+1 queries with eager loading
- Cache expensive query results
- Use connection pooling

### 2. Algorithm Efficiency
- Replace O(n²) with O(n log n) or O(n)
- Use appropriate data structures (hash map vs array)
- Avoid unnecessary computation in loops

### 3. Caching
- Cache expensive computations
- Cache external API responses
- Use CDN for static assets
- Implement Redis for distributed cache

### 4. Async/Parallel Processing
- Use async I/O for network/disk operations
- Parallelize independent operations
- Use worker threads/processes for CPU-intensive tasks

### 5. Memory Optimization
- Fix memory leaks
- Use streaming for large data
- Implement pagination
- Clean up event listeners

## Best Practices

✅ **DO:**
- Profile before optimizing (measure, don't guess)
- Focus on hot paths (80/20 rule)
- Use production-safe profilers in prod (py-spy, 0x)
- Profile with realistic data volumes
- Document baseline before optimizations
- Re-profile after each optimization
- Consider trade-offs (memory vs speed)
- Test edge cases (cold cache, high load)

❌ **DON'T:**
- Optimize without profiling first
- Micro-optimize code that runs rarely
- Profile with toy data (profile at scale)
- Ignore database query performance
- Profile only in development environment
- Change multiple things at once
- Sacrifice readability for negligible gains
- Forget to measure impact of changes

## Common Profiling Commands

### Python

```bash
# CPU profiling
python -m cProfile -s cumulative app.py

# Line-by-line profiling
pip install line_profiler
kernprof -l -v script.py

# Memory profiling
pip install memory_profiler
python -m memory_profiler script.py

# Production profiling (no overhead)
pip install py-spy
py-spy record -o profile.svg --pid 12345
```

### Node.js

```bash
# Flame graph
npm install -g 0x
0x app.js

# Clinic.js suite
clinic doctor -- node app.js
clinic flame -- node app.js
clinic bubbleprof -- node app.js

# Chrome DevTools
node --inspect app.js
# Open chrome://inspect
```

### Go

```bash
# CPU profile
go test -cpuprofile=cpu.prof
go tool pprof cpu.prof

# Memory profile
go test -memprofile=mem.prof
go tool pprof mem.prof

# Benchmark
go test -bench=. -benchmem
```

### Database

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;

-- MySQL
EXPLAIN FORMAT=JSON SELECT ...;

-- Check slow queries
SET long_query_time = 1;
SHOW VARIABLES LIKE 'slow_query%';
```

## Performance Metrics

### Response Time Targets

- **API endpoints**: < 200ms (p95)
- **Database queries**: < 50ms
- **Page load**: < 2s (First Contentful Paint)
- **Time to Interactive**: < 3.5s

### Resource Usage Targets

- **CPU**: < 70% average
- **Memory**: No leaks, stable over time
- **Database connections**: < 50% of pool
- **Cache hit rate**: > 90%

## Instructions

1. **Identify performance issue**
   - Slow endpoint, high resource usage, user complaints
   - Check production metrics/logs

2. **Reproduce locally**
   - Use production-like data volume
   - Simulate realistic load

3. **Choose profiling tool**
   - Python: cProfile or py-spy
   - Node.js: clinic.js or 0x
   - Go: pprof
   - Database: EXPLAIN ANALYZE

4. **Run profiler**
   - Profile the slow operation
   - Capture flame graph or profile data
   - Profile multiple times

5. **Analyze results**
   - Identify top 3 bottlenecks
   - Check for N+1 queries, memory leaks
   - Look for inefficient algorithms

6. **Prioritize fixes**
   - Estimate impact (time saved)
   - Start with database/algorithms
   - Quick wins first

7. **Optimize**
   - Make one change at a time
   - Test after each change
   - Re-profile to verify improvement

8. **Document**
   - Baseline metrics
   - Changes made
   - Performance improvement
   - Any trade-offs

## Constraints

- Must profile before optimizing (measure, don't guess)
- Must use production-safe profilers in production (py-spy, not cProfile)
- Must test with realistic data volumes (not toy data)
- Must document baseline metrics before changes
- Must re-profile after optimizations to verify improvement
- Must consider trade-offs (memory vs CPU, complexity vs performance)
- Should focus on hot paths (highest impact optimizations first)
- Should fix N+1 queries before micro-optimizations
- Must not sacrifice correctness for performance
- Must profile under realistic load conditions
