# Architecture Decision Records (ADRs)

This directory contains Architecture Decision Records - documents that capture important architectural decisions made for this project.

## What is an ADR?

An ADR is a document that captures an important architectural decision made along with its context and consequences. It provides:
- **What** decision was made
- **Why** it was made (context and rationale)
- **What** alternatives were considered
- **What** are the consequences (trade-offs)

## When to Create an ADR

Create an ADR when making decisions about:
- Technology choices (languages, frameworks, databases)
- Architecture patterns (monolith vs microservices, REST vs GraphQL)
- Significant refactorings or migrations
- Security or compliance approaches
- Cross-cutting concerns (logging, monitoring, auth)

**Don't create ADRs for:**
- Tactical implementation details
- Routine bug fixes
- Small code refactorings

## ADR Numbering

ADRs are numbered sequentially: ADR-0001, ADR-0002, etc.

The number never changes, even if the ADR is deprecated or superseded.

## ADR Status

- **Proposed:** Decision being considered
- **Accepted:** Decision approved and being implemented
- **Deprecated:** No longer relevant but kept for history
- **Superseded by ADR-XXX:** Replaced by a newer decision

## ADR Index

<!-- Add links to ADRs as they're created -->

### Active

*No ADRs yet - run `/init-project` to create the first one*

### Deprecated

*None yet*

## Creating a New ADR

1. Copy `docs/templates/ADR.template.md`
2. Name it `ADR-XXXX-title.md` (next sequential number)
3. Fill in all sections
4. Add to index above
5. Commit to repository

ADRs are never deleted - they provide a historical record of how the system evolved.
