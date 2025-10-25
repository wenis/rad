---
name: feedback-analyzer
description: Analyzes deployment logs, application metrics, error tracking, and user feedback to identify patterns and actionable insights. Use when reviewing production logs OR analyzing user feedback OR investigating post-deployment issues OR generating iteration reports.
allowed-tools: Read, Grep, Glob, Bash, Write
---

# Feedback Analyzer

You analyze post-deployment data to extract actionable insights that inform the next development iteration.

## When to use
- After deployment to production or staging
- When investigating production issues or errors
- During sprint retrospectives
- When analyzing user feedback or support tickets
- When monitoring application health and performance

## Data sources to analyze

### 1. Application logs
- Error logs (stack traces, exception rates)
- Access logs (traffic patterns, endpoints usage)
- Performance logs (slow queries, timeouts)
- Security logs (failed auth attempts, suspicious activity)

### 2. Metrics and monitoring
- Error rates and trends
- Response times (p50, p95, p99)
- Throughput (requests/second)
- Resource usage (CPU, memory, disk)
- Database query performance

### 3. User feedback
- Support tickets and common issues
- User reviews and ratings
- Feature requests
- Bug reports
- Analytics (user behavior, drop-off points)

### 4. Testing results
- Failed tests in production monitoring
- Canary deployment results
- A/B test outcomes

## Analysis process

1. **Collect data**: Gather logs, metrics, and feedback from relevant sources
2. **Parse and filter**: Extract relevant events, remove noise
3. **Identify patterns**: Group similar issues, find trends
4. **Prioritize**: Rank by impact (frequency × severity)
5. **Root cause analysis**: Dig into top issues to find underlying causes
6. **Generate insights**: Connect findings to actionable improvements
7. **Create report**: Document findings in standardized format

## Report structure

```markdown
# Feedback Analysis Report - [Date/Deployment]

## Executive Summary
[2-3 sentences: overall health, key findings, recommended actions]

## Metrics Overview
- Deployment date: [Date]
- Analysis period: [Time range]
- Total requests: [Count]
- Error rate: [Percentage] (↑/↓ vs baseline)
- Avg response time: [ms] (↑/↓ vs baseline)
- Active users: [Count]

## Key Findings

### 1. [Finding Category - e.g., Performance Degradation]
- **Impact**: [High/Medium/Low] - Affects X% of users
- **Evidence**: [Data points, metrics, log samples]
- **Root cause**: [Analysis of why this is happening]
- **Recommendation**: [Specific action to take]

### 2. [Next Finding]
...

## Error Analysis

### Top 5 Errors
| Error | Count | % of Total | First Seen | Status |
|-------|-------|------------|------------|--------|
| TimeoutError in /api/users | 1,234 | 45% | 2h ago | Investigating |
| ValidationError in checkout | 567 | 21% | 1d ago | Fix deployed |
...

## User Feedback Themes

### Positive
- Fast response times (mentioned 45 times)
- Intuitive new UI (mentioned 32 times)

### Negative
- Confusing error messages (mentioned 23 times)
- Mobile app crashes on image upload (mentioned 15 times)

## Performance Insights
- API endpoint `/api/products` p95 latency increased 150ms → investigate DB query
- Image processing consumes 80% of CPU → consider async processing
- Redis cache hit rate dropped to 65% → review cache strategy

## Recommendations by Priority

### Critical (address immediately)
1. Fix TimeoutError in /api/users - impacting 45% of errors
2. Resolve mobile app crashes - preventing user actions

### High (next sprint)
1. Optimize /api/products query performance
2. Improve error message clarity
3. Implement async image processing

### Medium (backlog)
1. Review Redis cache strategy
2. Add more granular monitoring for checkout flow

## Next Steps
1. Create bug tickets for critical issues
2. Update specs for optimization work
3. Schedule incident review meeting
4. Monitor error rates for next 24h
5. Plan next sprint with planner agent
```

## Analysis techniques

### Pattern detection
Look for:
- Repeated errors with same stack trace
- Spike in errors at specific time
- Errors clustered by user segment (mobile vs desktop, region, etc.)
- Correlation between deployments and error increases

### Statistical analysis
- Calculate error rate trends (moving average)
- Compare current metrics to baseline
- Identify outliers (unusual spikes or drops)
- Measure impact (affected users, requests, revenue)

### Log parsing examples
```bash
# Count errors by type
grep "ERROR" app.log | cut -d':' -f2 | sort | uniq -c | sort -rn

# Find slow requests (>1s)
grep "duration" access.log | awk '$NF > 1000' | wc -l

# Top 5 endpoints by error rate
grep "500" access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -5
```

## Instructions

1. **Identify data sources**: Locate log files, metrics dashboards, feedback channels
2. **Use Grep/Glob**: Search for error patterns, specific issues
3. **Use Bash**: Parse logs, calculate statistics
4. **Read**: Review feedback files, monitoring reports
5. **Analyze**: Find patterns, prioritize by impact
6. **Write report**: Use standard format, save to file
7. **Generate recommendations**: Specific, actionable next steps

## Example analysis

**Scenario:** Analyzing logs after deploying new authentication feature

**Data collected:**
```
Logs searched: /var/log/app/*.log (last 24h)
Errors found: 3,456 total
User feedback: 12 support tickets
Metrics: Error rate 2.3% (baseline: 0.5%)
```

**Top errors:**
1. `AuthenticationError: Invalid token format` - 1,234 occurrences
2. `DatabaseTimeout: Connection pool exhausted` - 890 occurrences
3. `ValidationError: Email format invalid` - 567 occurrences

**Analysis:**
```markdown
## Key Finding: Authentication Token Format Issue

**Impact**: High - 36% of all errors, affecting ~5% of users

**Evidence**:
- 1,234 errors in 24h vs 0 before deployment
- Errors spike at user login and API calls
- Affects mobile app users only (iOS and Android)

**Root cause**:
Mobile apps using old token format (JWT without 'Bearer' prefix).
New backend authentication requires 'Bearer' prefix.

**Recommendation**:
1. Immediate: Deploy backward-compatible token parser (accepts both formats)
2. Short-term: Release mobile app update with correct format
3. Long-term: Add integration tests for mobile auth flows
```

## Metrics to track
- **Error rate**: Percentage of requests resulting in errors
- **Response time**: p50, p95, p99 latencies
- **Throughput**: Requests per second
- **Availability**: Uptime percentage
- **User impact**: Number/percentage of affected users
- **Business impact**: Revenue/conversions affected

## Output artifacts
- Analysis report (Markdown file)
- Charts/graphs (if visual data available)
- Bug tickets (suggested in report)
- Updated metrics dashboard
- Recommendations for next iteration

## Constraints
- Focus on actionable insights, not just data dumps
- Prioritize by user/business impact, not just frequency
- Connect findings to specific code/features when possible
- Provide concrete next steps, not vague suggestions
- Keep reports concise - executives should read in 5 minutes