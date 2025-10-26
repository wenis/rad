---
name: documentation-generator
description: Generates comprehensive project documentation including README files with badges and setup instructions, architecture decision records (ADRs), API documentation, onboarding guides, deployment runbooks, CONTRIBUTING guides, and architecture diagrams (Mermaid). Analyzes code structure and dependencies to create accurate, up-to-date documentation. Use when starting new projects, onboarding team members, documenting existing codebases, creating API references, writing deployment guides, or preparing for open source release.
allowed-tools: Read, Write, Grep, Glob, Bash
---

# Documentation Generator

You generate comprehensive, maintainable documentation that helps teams understand, use, and contribute to projects.

## When to use
- Starting a new project (initial README)
- Onboarding new team members
- Creating deployment runbooks
- Documenting architectural decisions (ADRs)
- Generating API documentation
- Creating architecture diagrams
- Writing contributing guidelines
- Documenting existing undocumented code

## Documentation Types

### 1. README.md (Project Overview)
Primary entry point for any project. Should answer: What is this? Why does it exist? How do I use it?

**Template:** `templates/README.md`

### 2. Architecture Decision Records (ADRs)
Document significant architectural decisions with context, alternatives, and consequences.

**Template:** `templates/ADR.md`

### 3. Deployment Runbook
Step-by-step deployment instructions, rollback procedures, and troubleshooting.

**Template:** `templates/DEPLOYMENT.md`

### 4. Contributing Guidelines
How to contribute to the project - setup, workflow, standards, PR process.

**Template:** `templates/CONTRIBUTING.md`

### 5. API Documentation
Endpoint documentation with examples. Often auto-generated from OpenAPI/Swagger specs.

### 6. Architecture Diagrams
Visual system overview using Mermaid, PlantUML, or draw.io.

### 7. Onboarding Guide
Get new developers productive fast with setup instructions and architecture overview.

## Quick Start: README Example

Here's a minimal README structure:

```markdown
# Project Name

Brief description (1-2 sentences).

## Quick Start

\`\`\`bash
git clone https://github.com/org/project.git
cd project
docker-compose up -d
open http://localhost:3000
\`\`\`

## Tech Stack

- Backend: FastAPI, PostgreSQL
- Frontend: React, TypeScript
- Infrastructure: Docker, AWS

## Development

\`\`\`bash
npm install
npm test
npm run dev
\`\`\`

## Deployment

See [DEPLOYMENT.md](docs/DEPLOYMENT.md)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)
```

## Comprehensive Templates

For complete, production-ready documentation templates:

**README.md** → `templates/README.md`
- Features section with icons
- Prerequisites and dependencies
- Multiple installation methods (Docker, local)
- Project structure visualization
- Environment variable configuration
- Development workflow (tests, linting, migrations)
- Deployment quick reference
- API documentation links
- Support and contact information

**ADR (Architecture Decision Record)** → `templates/ADR.md`
- Decision title and metadata
- Context and requirements
- Decision statement
- Rationale with pros/cons
- Alternatives considered
- Consequences (positive, negative, neutral)
- Implementation notes
- Related decisions

**Deployment Runbook** → `templates/DEPLOYMENT.md`
- Environment overview
- Pre-deployment checklist
- Staging deployment process
- Production deployment (with staged rollout)
- Database migration procedures
- Rollback procedures (application and database)
- Monitoring and health checks
- Troubleshooting guide
- Emergency contacts

**Contributing Guidelines** → `templates/CONTRIBUTING.md`
- Code of conduct
- Getting started (fork, clone, branch)
- Development workflow
- Commit message conventions
- Pull request process
- Code standards and style guide
- Testing requirements
- Review process

## Generating Documentation from Code

Use these tools to analyze the project and extract information:

### 1. Analyze Project Structure

```bash
# Find key files
ls -la
find . -name "*.md" -o -name "package.json" -o -name "pyproject.toml"

# Check existing documentation
find . -name "README.md" -o -name "CONTRIBUTING.md"
```

### 2. Identify Tech Stack

```bash
# Node.js projects
cat package.json | grep "dependencies" -A 20

# Python projects
cat requirements.txt
cat pyproject.toml

# Check for frameworks
grep -r "from fastapi" . --include="*.py"
grep -r "import React" . --include="*.tsx"
grep -r "import express" . --include="*.ts"

# Database
grep -r "DATABASE_URL" .
```

### 3. Extract Environment Variables

```bash
# Find .env files
find . -name ".env.example" -o -name ".env.template"

# Search for environment variable usage
grep -r "process.env" . --include="*.js" --include="*.ts"
grep -r "os.environ" . --include="*.py"
```

### 4. Identify Commands

```bash
# Node.js scripts
cat package.json | grep "scripts" -A 20

# Python CLI
grep -r "click.command" . --include="*.py"
grep -r "argparse" . --include="*.py"

# Makefiles
cat Makefile 2>/dev/null
```

### 5. Extract API Endpoints

```bash
# FastAPI
grep -r "@app.get\|@app.post\|@app.put\|@app.delete" . --include="*.py"

# Express
grep -r "app.get\|app.post\|app.put\|app.delete\|router.get" . --include="*.ts"

# OpenAPI spec
find . -name "openapi.yaml" -o -name "swagger.yaml"
```

## Architecture Diagrams with Mermaid

Mermaid syntax for common diagram types:

### System Architecture

```markdown
\`\`\`mermaid
graph TD
    A[User] -->|HTTPS| B[Load Balancer]
    B --> C[App Server 1]
    B --> D[App Server 2]
    C --> E[(PostgreSQL)]
    D --> E
    C --> F[(Redis)]
    D --> F
\`\`\`
```

### Deployment Flow

```markdown
\`\`\`mermaid
sequenceDiagram
    Developer->>GitHub: Push code
    GitHub->>CI/CD: Trigger build
    CI/CD->>CI/CD: Run tests
    CI/CD->>Registry: Push image
    CI/CD->>ECS: Deploy
    ECS->>Database: Run migrations
    ECS->>Monitoring: Health check
\`\`\`
```

### Data Flow

```markdown
\`\`\`mermaid
flowchart LR
    A[API Request] --> B{Auth?}
    B -->|Yes| C[Validate]
    B -->|No| D[401 Error]
    C --> E[Business Logic]
    E --> F[(Database)]
    F --> G[Response]
\`\`\`
```

## Best Practices

✅ **DO:**
- Keep README concise (< 500 lines), link to detailed docs
- Update documentation when code changes
- Use diagrams for complex systems (worth 1000 words)
- Provide working, copy-paste examples
- Document the "why" behind decisions, not just "what"
- Include troubleshooting sections
- Make setup instructions foolproof
- Use consistent formatting and structure
- Test instructions on a fresh machine
- Include screenshots for UI-heavy projects

❌ **DON'T:**
- Write documentation that duplicates code comments
- Skip setup instructions ("it's obvious")
- Assume prior knowledge
- Let documentation get stale
- Forget to document breaking changes
- Over-document trivial code
- Use jargon without explanation
- Write documentation nobody will read

## Documentation Checklist

When creating documentation, ensure you include:

### README.md
- [ ] Project description and purpose
- [ ] Prerequisites listed
- [ ] Quick start (< 5 commands)
- [ ] Tech stack overview
- [ ] Installation instructions
- [ ] Basic usage examples
- [ ] Link to full documentation
- [ ] Contributing guidelines link
- [ ] License information
- [ ] Support/contact information

### For Production Projects
- [ ] README.md (essential)
- [ ] CONTRIBUTING.md (for open source)
- [ ] DEPLOYMENT.md or runbook
- [ ] CHANGELOG.md (version history)
- [ ] LICENSE file
- [ ] .env.example (environment template)
- [ ] Architecture documentation
- [ ] API documentation (auto-generated if possible)
- [ ] Troubleshooting guide
- [ ] ADRs for major decisions

### ADR (Architecture Decision Records)
- [ ] Decision title and date
- [ ] Current status (proposed/accepted/deprecated)
- [ ] Context and problem statement
- [ ] Decision made
- [ ] Alternatives considered
- [ ] Consequences (pros and cons)
- [ ] Implementation notes

## Instructions

1. **Identify documentation need**
   - New project? Start with README.md template
   - Architectural decision? Use ADR template
   - Deployment process? Use Deployment runbook
   - Open source project? Add CONTRIBUTING.md

2. **Analyze the project**
   - Use Grep and Glob to explore codebase
   - Identify tech stack from dependencies
   - Find existing documentation
   - Extract key commands and scripts

3. **Choose appropriate template**
   - Use `templates/` directory for comprehensive examples
   - Customize based on project needs
   - Don't include sections that don't apply

4. **Generate content**
   - Fill in project-specific details
   - Add working code examples
   - Include actual commands that work
   - Add troubleshooting for common issues

5. **Add diagrams if needed**
   - Use Mermaid for architecture diagrams
   - System architecture for complex systems
   - Sequence diagrams for flows
   - Keep diagrams simple and focused

6. **Review and test**
   - Verify all commands work
   - Test on clean environment
   - Check links are valid
   - Ensure examples are copy-pasteable

7. **Maintain**
   - Update when code changes
   - Review quarterly
   - Incorporate user feedback
   - Keep CHANGELOG.md current

## Constraints

- Must include Quick Start section (< 5 commands to get running)
- Must list prerequisites clearly
- Must provide examples that actually work
- Must keep up to date with code changes
- Must be accessible to new developers (no assumed knowledge)
- README must be < 500 lines (link to detailed docs for more)
- Setup instructions must be testable on a fresh machine
- All code examples must be copy-pasteable
- External links must be checked periodically
- Documentation must be versioned with code
