# [Project Name]

## Vision

**Problem:** [What problem does this solve?]

**Solution:** [How does this solve it?]

**Target Users:** [Who is this for?]

**Value Proposition:** [Why would users choose this?]

## Tech Stack

### Backend
- **Language:** [Python/JavaScript/Go/etc.]
- **Framework:** [FastAPI/Express/Django/etc.]
- **Database:** [PostgreSQL/MySQL/MongoDB/etc.]
- **Cache:** [Redis/Memcached/etc. if needed]
- **Queue:** [Celery/Bull/RabbitMQ/etc. if needed]

### Frontend (if applicable)
- **Framework:** [React/Vue/Angular/etc.]
- **Language:** [TypeScript/JavaScript]
- **State Management:** [Redux/Zustand/Context/etc.]

### Infrastructure
- **Hosting:** [AWS/GCP/Azure/Heroku/etc.]
- **Containers:** [Docker/Kubernetes/ECS/etc.]
- **CI/CD:** [GitHub Actions/GitLab CI/CircleCI/etc.]

### Key Tech Decisions
- **Why [Language]?** [Brief rationale]
- **Why [Database]?** [Brief rationale]
- **Why [Architecture Pattern]?** [Brief rationale]

See `docs/architecture/ADR-0001-initial-tech-stack.md` for detailed rationale.

## Architecture

**Pattern:** [Monolith/Modular Monolith/Microservices]

**API Style:** [REST/GraphQL/gRPC]

**Authentication:** [JWT/Sessions/OAuth]

**Key Principles:**
- [Principle 1: e.g., "API-first design"]
- [Principle 2: e.g., "Stateless services"]
- [Principle 3: e.g., "Event-driven where async needed"]

## Performance Targets

- **API Response Time:** p95 < [200ms]
- **Uptime:** [99.9%]
- **Concurrent Users:** [Support X concurrent users]
- **Data Volume:** [Handle X records]

## Security Requirements

- **Authentication:** [All endpoints require auth / Public + protected routes]
- **Authorization:** [Role-based access control / etc.]
- **Data Encryption:** [At rest: Yes/No, In transit: TLS]
- **Password Policy:** [Min 12 chars, bcrypt hashing]
- **Rate Limiting:** [100 requests/minute per user]
- **Compliance:** [GDPR/HIPAA/SOC2/None]

## Quality Standards

### Testing
- **Coverage Target:** 80% overall (critical paths 95%+)
- **Test Types:** Unit, Integration, E2E
- **CI/CD:** All tests must pass before merge

### Code Quality
- **Linting:** [eslint/pylint/gofmt]
- **Formatting:** [prettier/black/gofmt]
- **Type Safety:** [TypeScript/Python type hints/Go]

### Documentation
- **API Docs:** Auto-generated OpenAPI specs
- **Code Comments:** Only for complex logic
- **ADRs:** For architectural decisions

## Development Workflow

1. **Plan:** Create spec in `docs/specs/`
2. **Build:** Implement feature following specs
3. **Validate:** Run tests, security scan, performance check
4. **Deploy:** Staging → Production (with monitoring)
5. **Feedback:** Analyze metrics, iterate

See `.claude/PHILOSOPHY.md` for detailed workflow.

## Key Constraints

- [Constraint 1: e.g., "Must be GDPR compliant"]
- [Constraint 2: e.g., "Must support mobile browsers"]
- [Constraint 3: e.g., "Must integrate with existing auth system"]

## Team & Communication

- **Team Size:** [Current team size]
- **Communication:** [Slack/Discord/Email/etc.]
- **Issue Tracking:** [GitHub Issues/Jira/Linear/etc.]
- **Documentation:** This repo + [other locations]

## Next Steps

Ready to build features! Use:
- `/plan "feature description"` - Plan a feature
- `/rapid-dev "feature description"` - Full development cycle
- `/build`, `/validate`, `/ship` - Individual phases

---

*Last Updated:* [Date]
*Status:* [Active/Planning/Production]
