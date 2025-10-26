# Architecture Decision Record (ADR) Template

Use this template to document significant architectural decisions. Place ADRs in `docs/adr/` directory.

```markdown
# ADR-XXXX: [Decision Title]

**Date**: YYYY-MM-DD
**Status**: [Proposed | Accepted | Rejected | Deprecated | Superseded by ADR-YYYY]
**Deciders**: [Names and roles of people who made the decision]

## Context

What is the issue we're facing? What factors are driving this decision?

### Requirements
- Requirement 1
- Requirement 2
- Requirement 3

### Constraints
- Constraint 1
- Constraint 2

## Decision

We will [chosen solution in one sentence].

## Rationale

### Pros
- **Benefit 1**: Explanation
- **Benefit 2**: Explanation
- **Benefit 3**: Explanation

### Alternatives Considered

**Option 1: [Alternative Name]**
- Pros: [Benefits]
- Cons: [Drawbacks]
- Why not: [Reason for rejection]

**Option 2: [Alternative Name]**
- Pros: [Benefits]
- Cons: [Drawbacks]
- Why not: [Reason for rejection]

**Option 3: [Alternative Name]**
- Pros: [Benefits]
- Cons: [Drawbacks]
- Why not: [Reason for rejection]

## Consequences

### Positive
- Positive consequence 1
- Positive consequence 2

### Negative
- Negative consequence 1
- Negative consequence 2
- Mitigation: How we'll address this

### Neutral
- Neutral consequence 1
- Neutral consequence 2

## Implementation Notes

- Implementation detail 1
- Implementation detail 2
- Rollout strategy
- Migration plan (if applicable)

## Related Decisions

- ADR-XXXX: Related decision
- ADR-YYYY: Another related decision

## References

- [Link to documentation]
- [Link to discussion]
- [Link to research]
```

## Example: Database Selection

```markdown
# ADR-0001: Use PostgreSQL for Primary Database

**Date**: 2025-01-24
**Status**: Accepted
**Deciders**: Alice (Tech Lead), Bob (Backend Dev)

## Context

We need to choose a database for our application that handles user data, transactions, and relationships.

### Requirements
- ACID compliance for financial transactions
- Support for complex queries and joins
- Mature ecosystem and tooling
- Good performance for read-heavy workloads
- Support for JSON data (flexible schema)

## Decision

We will use **PostgreSQL 15** as our primary database.

## Rationale

### Pros
- **ACID compliance**: Critical for financial data integrity
- **Mature and stable**: 30+ years of development
- **Rich feature set**: JSON support, full-text search, geospatial
- **Performance**: Excellent for complex queries and analytical workloads
- **Ecosystem**: ORMs (SQLAlchemy, TypeORM), migration tools, monitoring
- **Open source**: No vendor lock-in, large community

### Alternatives Considered

**MySQL**
- Pros: Simpler, slightly better for write-heavy workloads
- Cons: Less feature-rich, weaker JSON support
- Why not: We need advanced features (CTEs, window functions)

**MongoDB**
- Pros: Flexible schema, horizontal scaling
- Cons: No transactions (at the time), eventual consistency
- Why not: Need ACID guarantees for financial data

**DynamoDB**
- Pros: Managed, auto-scaling, serverless
- Cons: Vendor lock-in, limited query flexibility, expensive for reads
- Why not: Complex queries required, want multi-cloud option

## Consequences

### Positive
- Reliable, well-tested database for critical data
- Rich querying capabilities (CTEs, window functions, JSON)
- Excellent tooling and community support
- Can use JSONB for semi-structured data

### Negative
- Vertical scaling (need bigger servers, not just add more)
- More complex to operate than managed services
- Requires expertise for performance tuning

### Neutral
- Need to set up replication for HA
- Will use connection pooling (PgBouncer)
- Regular backups and point-in-time recovery needed

## Implementation Notes

- Use Docker for local development
- RDS for staging/production (managed)
- Connection pooling with PgBouncer
- Read replicas for analytics queries
- Backup retention: 30 days
- Monitoring: CloudWatch + custom Prometheus metrics

## Related Decisions

- ADR-0002: Use SQLAlchemy as ORM
- ADR-0003: Database migration strategy (Alembic)

## References

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Comparison: PostgreSQL vs MySQL](https://example.com/comparison)
- Internal discussion: [Link to Slack thread]
```
