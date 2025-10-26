---
name: sync-docs
description: Scan codebase and sync SYSTEM.md to match current reality
---

# Sync Documentation

Scan the codebase and update `docs/SYSTEM.md` to reflect the current state of the project.

## Purpose

Over time, as features are built, the system evolves:
- New dependencies are added (Redis, GraphQL clients, etc.)
- New architectural patterns emerge (event-driven, caching layers)
- New infrastructure is introduced (CDNs, monitoring, queues)
- Performance characteristics change

`docs/SYSTEM.md` can drift out of sync with reality. This command detects drift and updates documentation to match the actual codebase.

## When to Use

Run `/sync-docs` when:
- Planner warns "SYSTEM.md may be outdated"
- After building several features
- Before major releases (ensure docs are current)
- When onboarding new developers (ensure accurate system overview)
- Periodically (e.g., monthly) as maintenance

## What This Does

The command will:
1. **Scan codebase for actual tech stack:**
   - Read dependency files (package.json, requirements.txt, go.mod, Cargo.toml, etc.)
   - List all current dependencies and frameworks
   - Identify major libraries and tools in use

2. **Compare to current SYSTEM.md:**
   - What's in the codebase but NOT in SYSTEM.md?
   - What's in SYSTEM.md but NOT in the codebase? (removed tech)
   - Are architecture patterns documented accurately?

3. **Present findings to user:**
   - Show what's missing from SYSTEM.md
   - Show what's outdated in SYSTEM.md
   - Recommend specific updates

4. **Update SYSTEM.md with user approval:**
   - Add new tech to Tech Stack section
   - Remove deprecated tech
   - Update architecture patterns if changed
   - Preserve original structure and format

## Instructions

**Step 1: Scan the codebase**

1. **Find and read dependency files:**
   - JavaScript/TypeScript: `package.json`, `yarn.lock`, `package-lock.json`
   - Python: `requirements.txt`, `Pipfile`, `pyproject.toml`, `setup.py`
   - Go: `go.mod`, `go.sum`
   - Rust: `Cargo.toml`
   - Ruby: `Gemfile`, `Gemfile.lock`
   - Java: `pom.xml`, `build.gradle`
   - .NET: `*.csproj`, `packages.config`

2. **Extract major dependencies:**
   - Focus on **major/notable** dependencies, not every single package
   - Examples of what to track:
     - Frameworks: Express, FastAPI, Django, Rails, Spring Boot
     - Databases: PostgreSQL, MongoDB, Redis, MySQL
     - Message queues: RabbitMQ, Kafka, Redis, SQS
     - Authentication: Passport, Auth0, JWT libraries
     - Testing: Jest, Pytest, RSpec, JUnit
     - Infrastructure: Docker, Kubernetes, AWS SDK, GCP SDK
   - **Ignore trivial utilities** (lodash, uuid, etc.) - focus on architectural decisions

3. **Scan code for architectural patterns:**
   - Look for common patterns:
     - REST API endpoints (Express routes, FastAPI routers, etc.)
     - GraphQL schemas
     - WebSocket connections
     - Event emitters/subscribers
     - Caching mechanisms
     - Background jobs/workers
   - Use Grep to find key patterns (e.g., `@app.route`, `addEventListener`, etc.)

**Step 2: Read current SYSTEM.md**

1. **Read `docs/SYSTEM.md`:**
   - Extract current Tech Stack section
   - Extract current Architecture section
   - Note what's documented

**Step 3: Compare and identify drift**

1. **Categorize findings:**
   - **NEW tech** (in codebase, not in SYSTEM.md):
     - Example: Found Redis in package.json, but SYSTEM.md doesn't mention Redis
   - **REMOVED tech** (in SYSTEM.md, not in codebase):
     - Example: SYSTEM.md mentions MongoDB, but no MongoDB dependencies found
   - **CHANGED patterns** (code shows different architecture than SYSTEM.md):
     - Example: SYSTEM.md says REST API, but found GraphQL schema files

2. **Create summary report for user:**
   ```markdown
   ## Documentation Sync Analysis

   ### Tech Stack Drift

   **New dependencies detected (not in SYSTEM.md):**
   - Redis (version 4.6.0) - likely for caching
   - Bull (version 4.10.0) - job queue for background tasks
   - Passport (version 0.6.0) - authentication middleware

   **Tech in SYSTEM.md but not found in codebase:**
   - MongoDB - no MongoDB dependencies or connection code found

   **Architecture Pattern Changes:**
   - SYSTEM.md says "REST API" but found GraphQL schema at `src/schema.graphql`
   - Found WebSocket server setup in `src/sockets/index.ts` (not documented)

   ### Recommended Updates

   **Tech Stack section:**
   - Add: Redis 4.6.0 (cache)
   - Add: Bull 4.10.0 (job queue)
   - Add: Passport 0.6.0 (authentication)
   - Remove: MongoDB (no longer used)

   **Architecture section:**
   - Update: "REST API with GraphQL for complex queries"
   - Add: "WebSocket server for real-time updates"
   - Add: "Background job processing with Bull queue"
   ```

**Step 4: Get user approval**

Use AskUserQuestion tool:
- "Should I update SYSTEM.md with these changes?"
- Options: "Yes, update all", "Let me review each", "No, SYSTEM.md is correct as-is"

**Step 5: Update SYSTEM.md**

If approved:
1. **Update Tech Stack section:**
   - Add new dependencies in appropriate subsections (Backend, Frontend, Infrastructure)
   - Remove deprecated tech
   - Keep formatting consistent with current structure

2. **Update Architecture section:**
   - Add new patterns discovered
   - Update descriptions to match reality
   - Keep it concise (SYSTEM.md should stay 1-2 pages)

3. **Preserve other sections:**
   - Don't change Vision, Performance Targets, Security Requirements unless user explicitly requests
   - Only update what's verifiably different

4. **Report back:**
   - "Updated SYSTEM.md with [N] changes"
   - List what was added/removed
   - Suggest: "Review docs/SYSTEM.md to verify accuracy"

## Example Usage

**User runs:** `/sync-docs`

**What happens:**
1. Scans package.json, finds Redis, Bull, Passport
2. Reads SYSTEM.md, sees MongoDB listed but Redis not mentioned
3. Greps codebase, finds GraphQL schema files
4. Presents findings to user
5. User approves updates
6. Updates SYSTEM.md Tech Stack and Architecture sections
7. Reports: "Updated SYSTEM.md - added Redis, Bull, Passport; removed MongoDB; noted GraphQL usage"

## Output Format

After sync, provide:
```markdown
✅ Documentation sync complete!

**Updated `docs/SYSTEM.md`:**

**Added to Tech Stack:**
- Redis 4.6.0 - Caching layer
- Bull 4.10.0 - Background job queue
- Passport 0.6.0 - Authentication middleware

**Removed from Tech Stack:**
- MongoDB (no longer in use)

**Updated Architecture:**
- API style: REST + GraphQL for complex queries
- Added: WebSocket server for real-time updates
- Added: Background job processing with Bull

**Next steps:**
- Review `docs/SYSTEM.md` to verify accuracy
- Consider creating an ADR for major changes (e.g., adding GraphQL)
- All future features will now reference updated tech stack
```

## Constraints

- **Only update what's verifiable** - don't guess or infer too much
- **Focus on major dependencies** - ignore trivial utilities
- **Preserve SYSTEM.md structure** - don't reformat or reorganize
- **Get user approval** - don't auto-update without confirmation
- **Stay concise** - SYSTEM.md should remain 1-2 pages, not a full inventory
- **If major changes detected** - recommend creating ADRs to document why

## Edge Cases

### If SYSTEM.md doesn't exist
```
"No docs/SYSTEM.md found. Run `/init-project` first to create system documentation."
```

### If no drift detected
```
"✅ SYSTEM.md is up-to-date! No changes needed.

The current tech stack matches what's in the codebase."
```

### If massive drift (10+ new dependencies)
```
"⚠️ Significant drift detected - found 15+ new dependencies.

This suggests SYSTEM.md hasn't been updated in a while.

Recommendation: Review each change carefully, and consider:
1. Are all these dependencies intentional?
2. Should we document major architectural decisions in ADRs?
3. Is the system growing too complex? (tech sprawl)

Proceed with sync? [Yes / Let me review the list first]"
```

## Notes

- This command is **read-heavy** (scans files, compares) and **write-light** (only updates SYSTEM.md)
- The goal is to keep SYSTEM.md **accurate**, not **exhaustive**
- When in doubt, ask user for clarification rather than guessing
- SYSTEM.md is the source of truth for **intended** architecture; code is the source of truth for **actual** implementation
- This command helps align the two
