---
name: commit-message-formatter
description: Formats git commit messages following Conventional Commits standard (feat, fix, docs, refactor, test, chore). Analyzes staged changes (git diff) to determine commit type, scope, and description. Generates semantic, well-structured messages compatible with semantic versioning and changelog generation. Includes breaking change notifications and references to issues. Use when creating commits, enforcing commit message standards, enabling automatic changelog generation, or preparing for semantic releases.
allowed-tools: Read, Grep, Bash
---

# Commit Message Formatter

You generate semantic, well-structured git commit messages following the Conventional Commits standard.

## When to use
- Before creating a commit
- When reviewing/improving commit messages
- When enforcing commit standards in CI
- Teaching team about good commit messages

## Conventional Commits Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type (Required)
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Formatting (no code change)
- `refactor`: Code restructure (no behavior change)
- `perf`: Performance improvement
- `test`: Add/update tests
- `chore`: Maintenance, dependencies

### Scope (Optional)
Area of codebase affected: `auth`, `api`, `ui`, `db`, etc.

### Subject (Required)
- Imperative mood: "add" not "added"
- No capital first letter
- No period at end
- Max 72 characters

### Body (Optional)
- Explain "what" and "why", not "how"
- Wrap at 72 characters
- Blank line after subject

### Footer (Optional)
- Breaking changes: `BREAKING CHANGE: description`
- Issue references: `Closes #123`, `Fixes #456`

## Examples

### Feature Addition
```
feat(auth): add OAuth2 authentication

Implement OAuth2 flow using authorization code grant.
Users can now sign in with Google or GitHub accounts.

- Added OAuth2 client configuration
- Created callback endpoint for token exchange
- Updated user model to store provider info

Closes #45
```

### Bug Fix
```
fix(api): resolve database connection timeout

Connection pool was exhausting under load, causing
timeout errors for users. Increased pool size from
10 to 50 connections and added connection recycling.

Fixes #123
```

### Breaking Change
```
feat(api): change user endpoint response format

BREAKING CHANGE: User endpoint now returns ISO 8601
timestamps instead of Unix timestamps.

Migration guide:
- Old: { "createdAt": 1642425600 }
- New: { "createdAt": "2022-01-17T12:00:00Z" }
```

### Simple Changes
```
docs: update README with installation steps
```

```
test: add integration tests for payment flow
```

```
chore: upgrade dependencies to latest versions
```

```
refactor(ui): simplify state management logic
```

## Analysis Process

1. **Examine changes:**
   ```bash
   # See staged changes
   git diff --cached

   # See changed files
   git diff --cached --name-only

   # See commit stats
   git diff --cached --stat
   ```

2. **Categorize change:**
   - New functionality → `feat`
   - Fixing broken behavior → `fix`
   - Code improvement (no behavior change) → `refactor`
   - Performance optimization → `perf`
   - Documentation → `docs`
   - Tests → `test`
   - Build/dependencies → `chore`

3. **Determine scope:**
   - Single module/area → use scope (e.g., `auth`, `api`)
   - Multiple areas → omit scope or use general scope
   - Frontend/backend distinction → `fe`/`be` or `ui`/`api`

4. **Write subject:**
   - Start with verb: "add", "fix", "update", "remove", "refactor"
   - Be specific: "fix login redirect" not "fix bug"
   - Keep concise: under 72 chars

5. **Add body (if needed):**
   - Why was this change necessary?
   - What problem does it solve?
   - Any important implementation details?
   - Breaking changes or migration notes

6. **Add footer (if applicable):**
   - Reference issues: `Closes #123`
   - Breaking changes: `BREAKING CHANGE: ...`
   - Co-authors: `Co-authored-by: Name <email>`

## Implementation

```bash
# Analyze staged changes
git diff --cached --stat
git diff --cached

# Generate commit message based on changes
# Save to file for editing
git commit  # Opens editor with generated message
```

## Common Patterns

### Multiple Related Changes
```
feat(api): add user profile endpoints

- GET /api/users/:id/profile - Retrieve profile
- PUT /api/users/:id/profile - Update profile
- POST /api/users/:id/avatar - Upload avatar

All endpoints require authentication and validate
permissions before allowing access.

Closes #67
```

### Dependency Update
```
chore(deps): upgrade React to v18.2.0

Includes React DOM and testing library updates.
No breaking changes for our usage.
```

### Security Fix
```
fix(auth): prevent JWT token reuse after logout

Tokens are now invalidated server-side when users
log out, preventing reuse if intercepted.

This addresses a security vulnerability where
logged-out tokens could still access protected
endpoints.

Security: Fixes CVE-2024-1234
```

### Documentation
```
docs(api): add OpenAPI spec examples

Added request/response examples for all user
endpoints to improve API documentation clarity.
```

### Refactoring
```
refactor(db): extract query builders to repository layer

Moved SQL query construction from controllers to
repository classes for better separation of concerns
and testability.
```

## Bad Examples (Don't Do This)

❌ **Too vague:**
```
fix: bug fix
update: changes
```

❌ **Past tense:**
```
feat: added user authentication
fixed: resolved database issue
```

❌ **Capitalized:**
```
Feat: Add user authentication
```

❌ **Period at end:**
```
feat: add user authentication.
```

❌ **Too long subject:**
```
feat: add comprehensive user authentication system with OAuth2, JWT tokens, and session management
```

❌ **Missing type:**
```
add user authentication
```

## Validation Rules

- [ ] Type is valid (feat/fix/docs/style/refactor/perf/test/chore)
- [ ] Subject starts with lowercase letter
- [ ] Subject is in imperative mood
- [ ] Subject is under 72 characters
- [ ] Subject does not end with period
- [ ] Body lines wrap at 72 characters
- [ ] Blank line between subject and body
- [ ] Breaking changes noted in footer
- [ ] Issue references use correct format

## Tools Integration

### commitlint
```bash
npm install -D @commitlint/cli @commitlint/config-conventional

# commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
};

# Add to package.json
"husky": {
  "hooks": {
    "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
  }
}
```

### commitizen
```bash
npm install -D commitizen cz-conventional-changelog

# Setup
npx commitizen init cz-conventional-changelog --save-dev --save-exact

# Use instead of git commit
npx cz
```

## Instructions

1. **Read changes:** Use `git diff --cached` to see what's changing
2. **Categorize:** Determine type (feat/fix/etc.) and scope
3. **Write subject:** Imperative, lowercase, under 72 chars
4. **Add body:** Explain why (if non-obvious)
5. **Add footer:** Reference issues, note breaking changes
6. **Validate:** Check against rules above
7. **Format:** Ensure proper structure and line wrapping

## Best Practices

✅ **DO:**
- Use imperative mood ("add" not "added")
- Be specific in subject
- Explain "why" in body
- Reference issues in footer
- Keep commits atomic (one logical change)
- Write commits as you go, not at the end

❌ **DON'T:**
- Be vague ("fix bug", "update code")
- Use past tense
- Capitalize subject
- Make commits too large
- Forget to reference issues
- Skip the body for non-trivial changes

## Output Format

When generating commit message, output in this format:

```
<type>(<scope>): <subject>

<body paragraph 1>

<body paragraph 2>

<footer>
```

Then explain:
- Why this type was chosen
- What the commit accomplishes
- Any important context

## Constraints

- Must follow Conventional Commits specification
- Subject must be under 72 characters
- Must use imperative mood
- Must not capitalize subject
- Must not end subject with period
- Breaking changes must be noted in footer
