---
name: init-project
description: Initialize project with system overview, tech stack, and architectural foundation. Run this FIRST before building any features to establish system context.
---

# Initialize Project

Create the foundational system documentation that all agents will reference when planning, building, validating, and deploying features.

## Purpose

Before vibe coding features, establish:
- **What** you're building (problem, users, goals)
- **How** you'll build it (tech stack, architecture)
- **Why** key decisions were made (rationale)
- **Standards** to follow (performance, security)

This prevents architectural drift and ensures consistency as the system grows.

## When to Use

Run this **BEFORE** your first feature when:
- Starting a new project from scratch
- No `docs/SYSTEM.md` exists
- Need to establish architectural foundation

**DO NOT run this** if:
- System already initialized (docs/SYSTEM.md exists)
- You're just adding a feature to existing system

## What This Creates

1. **`README.md`** (project root) - Project overview for newcomers
2. **`docs/SYSTEM.md`** - Single-page system overview (tech stack, architecture, requirements)
3. **`docs/architecture/`** - Directory for ADRs (Architecture Decision Records)
4. **`docs/architecture/ADR-0001-initial-tech-stack.md`** - First ADR documenting tech choices
5. **`docs/standards/`** - Directory for coding standards (created later as needed)

## Instructions

### Phase 1: Gather System Context

Interview the user to understand:

1. **Problem & Purpose**
   - What problem does this solve?
   - Who is the target user?
   - What's the core value proposition?
   - What makes this different/better?

2. **Tech Stack Preferences**
   - Any preferred languages? (Python, JavaScript, Go, etc.)
   - Any preferred frameworks? (FastAPI, Express, Django, NestJS, etc.)
   - Database preference? (PostgreSQL, MySQL, MongoDB, etc.)
   - Infrastructure preference? (AWS, GCP, Azure, Docker, etc.)
   - If no preferences, recommend based on:
     - Team expertise
     - Problem domain
     - Scale requirements
     - Time to market needs

3. **Architecture Approach**
   - Start as monolith or microservices?
   - API style: REST, GraphQL, gRPC?
   - Auth approach: JWT, sessions, OAuth?
   - Suggest: "Start simple, refactor when needed"

4. **Non-Functional Requirements**
   - Expected users: 100? 10k? 1M?
   - Performance targets: API response time? Uptime?
   - Security requirements: Compliance? Data sensitivity?
   - Deployment frequency: Daily? Weekly?

5. **Key Constraints**
   - Budget limitations?
   - Time constraints?
   - Compliance requirements? (GDPR, HIPAA, SOC2)
   - Integration requirements? (existing systems)

### Phase 2: Conventions & Standards

Ask user:
```
"Would you like to include opinionated coding conventions?
These include:
- Git commit format (Conventional Commits)
- API standards (REST best practices, error formats)
- Code quality (automated formatting, linting)
- Testing requirements (80% coverage)
- Security standards (auth, rate limiting)

Default: Yes (recommended for consistency)
You can modify/remove any conventions later."
```

If **Yes** (default):
- Copy `.claude/templates/CONVENTIONS.md` to `docs/CONVENTIONS.md`
- These become the team's coding standards

If **No**:
- Skip conventions file
- Team will define standards as needed

### Phase 3: Create System Documentation

1. **Create root `README.md`:**
   - Project-level README for the repository
   - Brief description of what the project is
   - Links to documentation
   - Getting started instructions
   - Keep concise (< 1 page)

2. **Create `docs/SYSTEM.md`:**
   - Use template from `docs/templates/SYSTEM.template.md`
   - Fill in all sections based on user responses
   - Keep to 1-2 pages maximum
   - Focus on **what** and **why**, not **how**
   - If conventions included, add note: "See `docs/CONVENTIONS.md` for coding standards"

3. **Create first ADR:**
   - `docs/architecture/ADR-0001-initial-tech-stack.md`
   - Document why each tech choice was made
   - Include alternatives considered
   - Note trade-offs accepted

4. **Create directory structure:**
   ```
   docs/
     SYSTEM.md                          # System overview
     CONVENTIONS.md                     # Coding standards (if opted in)
     architecture/
       ADR-0001-initial-tech-stack.md  # Tech choices
       README.md                        # ADR index
     specs/                             # Feature specs (created by planner)
     validation/                        # Test reports (created by validator)
     deployment/                        # Deploy reports (created by shipper)
     feedback/                          # Feedback analysis (created by shipper)
   ```

### Phase 4: Report Back

Provide user with:
- Summary of system overview
- Paths to created files:
  - README.md (project overview)
  - docs/SYSTEM.md (system details)
  - docs/CONVENTIONS.md (coding standards, if included)
  - docs/architecture/ADR-0001-*.md (tech decisions)
- Confirmation that agents will now reference system context
- If conventions included: "Agents will follow coding standards in docs/CONVENTIONS.md"
- Suggestion: "Ready to build first feature - run `/plan` or `/rapid-dev`"

## Templates

### Root README.md Template

```markdown
# [Project Name]

[Brief 2-3 sentence description of what this project does and who it's for]

## Overview

**Problem:** [What problem does this solve?]

**Solution:** [How does this project solve it?]

**Target Users:** [Who is this for?]

## Tech Stack

- **Backend:** [Language + Framework] (e.g., Python + FastAPI)
- **Frontend:** [Framework] (e.g., React + TypeScript) [if applicable]
- **Database:** [Database] (e.g., PostgreSQL)
- **Infrastructure:** [Platform] (e.g., AWS ECS, Docker)

See `docs/SYSTEM.md` for detailed architecture and requirements.

## Getting Started

### Prerequisites

- [Language/Runtime] version X.X
- [Database] version X.X
- [Other tools needed]

### Installation

```bash
# Clone the repository
git clone [repository-url]

# Install dependencies
[installation commands]

# Set up environment
cp .env.example .env
# Edit .env with your values

# Run migrations (if applicable)
[migration commands]

# Start the application
[start commands]
```

### Running Tests

```bash
[test commands]
```

## Development

This project uses [Claude Code's Rapid Agile Development workflow](https://claude.com/claude-code):

- **Plan features:** `/plan "feature description"`
- **Build features:** `/rapid-dev "feature description"`
- **Individual phases:** `/build`, `/validate`, `/ship`

See `.claude/README.md` for complete workflow documentation.

## Documentation

- `README.md` (this file) - Project overview and getting started
- `docs/SYSTEM.md` - System architecture, tech stack, requirements
- `docs/architecture/` - Architecture Decision Records (ADRs)
- `docs/specs/` - Feature specifications
- `docs/standards/` - Coding standards and conventions
- `.claude/` - Workflow system and agent configuration

## Project Structure

```
[project-name]/
  README.md                 # This file
  [src/main code directory]/
  [tests/]
  docs/                     # System documentation
  .claude/                  # Workflow configuration
```

## Contributing

[Add contribution guidelines if applicable]

## License

[Add license information]

---

**Status:** [Planning/Active Development/Production/Maintenance]

**Last Updated:** [Date]
```

### SYSTEM.md Template

```markdown
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
- **Cache:** [Redis/Memcached/etc.]
- **Queue:** [Celery/Bull/RabbitMQ/etc. if needed]

### Frontend (if applicable)
- **Framework:** [React/Vue/Angular/etc.]
- **Language:** [TypeScript/JavaScript]
- **State Management:** [Redux/Zustand/etc.]

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
- **Authorization:** [Role-based access control]
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
```

### ADR Template

```markdown
# ADR-[NUMBER]: [Title - Decision Made]

## Status
[Proposed/Accepted/Deprecated/Superseded by ADR-XXX]

**Date:** [YYYY-MM-DD]

## Context

[What is the issue we're trying to solve? Include:]
- Business/technical problem
- Constraints we're operating under
- Requirements that influenced decision
- Forces at play

## Decision

[What did we decide to do?]

[Be specific and concrete. Include:]
- Exact technology/pattern chosen
- How it will be implemented
- Key aspects of the approach

## Rationale

[Why did we make this decision?]

[Explain the reasoning:]
- What makes this the best choice?
- How does it solve the problem?
- What benefits does it provide?

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]
- [Benefit 3]

### Negative
- [Trade-off 1]
- [Trade-off 2]
- [Limitation 1]

### Neutral
- [Change 1 to accommodate]
- [Learning curve for team]

## Alternatives Considered

### [Alternative 1 Name]
**Pros:** [Benefits]
**Cons:** [Drawbacks]
**Why not chosen:** [Reason]

### [Alternative 2 Name]
**Pros:** [Benefits]
**Cons:** [Drawbacks]
**Why not chosen:** [Reason]

## References

- [Link to relevant discussion]
- [Documentation URL]
- [Related ADRs]

## Notes

[Any additional context, follow-up items, or implementation notes]

---

*Author:* [Name]
*Reviewers:* [Names]
```

## Example: Complete Initialization

**User input:** "I want to build a SaaS platform for team task management"

**System creates:**

**`README.md`:**
```markdown
# TaskFlow - Team Task Management SaaS

A simple, fast, collaborative task management platform for small to medium teams.

## Overview

**Problem:** Teams struggle with scattered task management across email, Slack, and spreadsheets.

**Solution:** TaskFlow provides centralized task management with real-time collaboration, helping teams stay organized and productive.

**Target Users:** Small to medium teams (5-50 people) looking for simple, effective task management.

## Tech Stack

- **Backend:** Python 3.11 + FastAPI
- **Frontend:** React 18 + TypeScript
- **Database:** PostgreSQL 15
- **Cache:** Redis
- **Infrastructure:** AWS ECS + Docker

See `docs/SYSTEM.md` for detailed architecture and requirements.

## Getting Started

### Prerequisites

- Python 3.11+
- Node.js 18+
- PostgreSQL 15
- Redis 7
- Docker (for containerized development)

### Installation

```bash
# Clone the repository
git clone https://github.com/yourorg/taskflow.git

# Install backend dependencies
cd backend
pip install -r requirements.txt

# Install frontend dependencies
cd ../frontend
npm install

# Set up environment
cp .env.example .env
# Edit .env with your configuration

# Run database migrations
cd ../backend
alembic upgrade head

# Start the application
# Backend:
uvicorn main:app --reload

# Frontend (in another terminal):
cd frontend
npm run dev
```

### Running Tests

```bash
# Backend tests
cd backend
pytest

# Frontend tests
cd frontend
npm test
```

## Development

This project uses Claude Code's Rapid Agile Development workflow:

- **Plan features:** `/plan "feature description"`
- **Build features:** `/rapid-dev "feature description"`
- **Individual phases:** `/build`, `/validate`, `/ship`

See `.claude/README.md` for complete workflow documentation.

## Documentation

- `README.md` (this file) - Project overview and getting started
- `docs/SYSTEM.md` - System architecture, tech stack, requirements
- `docs/architecture/` - Architecture Decision Records (ADRs)
- `docs/specs/` - Feature specifications
- `.claude/` - Workflow system and agent configuration

## Project Structure

```
taskflow/
  README.md                 # This file
  backend/                  # FastAPI backend
  frontend/                 # React frontend
  docs/                     # System documentation
  .claude/                  # Workflow configuration
```

---

**Status:** Active Development

**Last Updated:** 2025-01-15
```

**`docs/SYSTEM.md`:**
```markdown
# TaskFlow - Team Task Management SaaS

## Vision
**Problem:** Teams struggle with scattered task management across email, Slack, and spreadsheets.
**Solution:** Centralized task management with real-time collaboration.
**Target Users:** Small to medium teams (5-50 people).
**Value Proposition:** Simple, fast, collaborative task management.

## Tech Stack
### Backend
- **Language:** Python 3.11
- **Framework:** FastAPI
- **Database:** PostgreSQL 15
- **Cache:** Redis
- **Queue:** Celery (for async notifications)

### Frontend
- **Framework:** React 18
- **Language:** TypeScript
- **State Management:** Zustand

### Infrastructure
- **Hosting:** AWS (ECS)
- **Containers:** Docker
- **CI/CD:** GitHub Actions

## Architecture
**Pattern:** Modular Monolith (can split later if needed)
**API Style:** REST (JSON)
**Authentication:** JWT tokens

## Performance Targets
- API p95 < 200ms
- Uptime: 99.9%
- Support 10k concurrent users

## Security Requirements
- All endpoints require authentication
- Password: bcrypt, min 12 chars
- Rate limiting: 100 req/min per user
- GDPR compliant
```

**`docs/architecture/ADR-0001-initial-tech-stack.md`:**
```markdown
# ADR-0001: Choose Python + FastAPI + PostgreSQL

## Status
Accepted (2025-01-15)

## Context
Need to choose tech stack for task management SaaS. Requirements:
- Fast development (small team)
- Good performance (real-time updates)
- Easy to hire for (common stack)
- Can scale to 10k users

## Decision
- Backend: Python + FastAPI
- Database: PostgreSQL
- Frontend: React + TypeScript

## Rationale
- FastAPI: Modern, fast, auto-generates OpenAPI docs
- Python: Team has expertise, rich ecosystem
- PostgreSQL: Proven, handles complex queries, JSONB for flexibility
- React: Industry standard, large talent pool

## Consequences
### Positive
+ Fast development with Python
+ Auto-generated API docs
+ Strong typing (FastAPI + TypeScript)
+ Proven at scale

### Negative
- Python slower than Go/Rust (acceptable for our scale)
- More moving parts than Rails/Django

## Alternatives Considered
### Node.js + Express
**Pros:** JavaScript everywhere, fast I/O
**Cons:** Less structured than FastAPI, no auto docs
**Why not:** Team prefers Python

### Ruby on Rails
**Pros:** Very fast development, conventions
**Cons:** Declining popularity, slower runtime
**Why not:** Harder to hire for
```

## Edge Cases

### If SYSTEM.md Already Exists
```
User runs: /init-project

Response: "System already initialized! Found docs/SYSTEM.md

If you want to:
- Update system docs → Edit docs/SYSTEM.md manually
- Document a new architectural decision → Create ADR manually
- Start fresh → Delete docs/SYSTEM.md first (not recommended)

Ready to build features:
- /plan 'feature description'
- /rapid-dev 'feature description'
```

### If User Doesn't Know Tech Stack
```
Planner: "No problem! Let me recommend based on your requirements.

You said: Small team, need to ship fast, scale to 10k users

I recommend:
- FastAPI (Python) - Fast to build, scales well, auto-generates docs
- PostgreSQL - Reliable, flexible, proven
- React - Industry standard, easy to hire

Sound good? Or do you have preferences?"
```

## Success Criteria

After `/init-project`, the project should have:
- ✅ `README.md` created (project overview)
- ✅ `docs/SYSTEM.md` created (1-2 pages)
- ✅ `docs/architecture/ADR-0001-*.md` created
- ✅ Directory structure established
- ✅ All agents will now read system context before working
- ✅ New developers can read README.md → understand project immediately
- ✅ User understands next steps (build features)

## Constraints

- Must complete in < 15 minutes (user interview + doc creation)
- SYSTEM.md must fit on 1-2 pages
- Must create first ADR (establishes pattern)
- Must be actionable immediately (can build features right after)
