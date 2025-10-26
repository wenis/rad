---
name: pr-description-generator
description: Generates comprehensive GitHub/GitLab PR descriptions by analyzing git diff, commit messages, and code changes. Creates structured descriptions with summary of changes, testing plan, deployment notes, breaking changes, related issues, and reviewer checklists. Follows PR templates and includes screenshots for UI changes. Use when creating pull requests, documenting complex changes, ensuring thorough code reviews, standardizing PR format, or preparing for deployment approvals.
allowed-tools: Read, Grep, Bash
---

# PR Description Generator

You generate comprehensive, structured pull request descriptions that help reviewers understand changes quickly.

## When to use
- Before creating a pull request
- When updating an existing PR
- Teaching team about good PR descriptions
- Automating PR creation in CI/CD

## PR Description Template

```markdown
## Description

Brief summary of what this PR does and why.

## Type of Change

- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to break)
- [ ] Documentation update
- [ ] Refactoring (no functional changes)
- [ ] Performance improvement
- [ ] Dependency update

## What Changed

- Bullet point summary of key changes
- Another important change
- One more change

## Why

Explain the motivation:
- What problem does this solve?
- Why is this approach better?
- Link to issue/discussion if applicable

## Testing

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Tested manually

**Manual testing steps:**
1. Step to reproduce/test
2. Expected behavior
3. Actual behavior

**Test coverage:** X% (↑/↓ from baseline)

## Screenshots/Videos (if applicable)

Before:
[screenshot or N/A]

After:
[screenshot or N/A]

## Breaking Changes

- [ ] Yes (describe below)
- [x] No

**If yes, describe:**
- What breaks?
- Migration path for users?
- When will old code be removed?

## Performance Impact

- [ ] Improves performance
- [ ] No performance impact
- [ ] May impact performance (explain below)

**Benchmarks:**
- Before: X ms
- After: Y ms
- Improvement: Z%

## Security Considerations

- [ ] Reviewed for security issues
- [ ] No new security vulnerabilities
- [ ] Security review needed

**Notes:**
[Any security-related changes or concerns]

## Deployment Notes

- [ ] Requires database migrations
- [ ] Requires environment variable changes
- [ ] Requires configuration changes
- [ ] No special deployment needs

**Instructions:**
[Any special deployment steps]

## Checklist

- [ ] Code follows project conventions
- [ ] Self-reviewed the code
- [ ] Added/updated documentation
- [ ] Added/updated tests
- [ ] All tests passing
- [ ] No new linter warnings
- [ ] Updated CHANGELOG (if applicable)

## Related Issues/PRs

Closes #123
Related to #456
Depends on #789

## Reviewer Notes

- Focus review on [specific area]
- Known issue: [describe any known issues]
- Future work: [what's left for future PRs]

---

**Estimated review time:** 15 minutes
```

## Analysis Process

1. **Get PR context:**
   ```bash
   # Current branch and commits
   git log main..HEAD --oneline

   # Files changed
   git diff main...HEAD --name-only

   # Stats
   git diff main...HEAD --stat

   # Full diff
   git diff main...HEAD
   ```

2. **Analyze commit messages:**
   - Extract feat/fix/refactor from commit types
   - Identify scope and affected areas
   - Build timeline of changes

3. **Categorize change type:**
   - All `feat` commits → New feature
   - All `fix` commits → Bug fix
   - Mixed → Multiple changes
   - `refactor` → Refactoring
   - `perf` → Performance improvement

4. **Identify key changes:**
   - What files were modified?
   - What modules/features affected?
   - What's the scope of changes?

5. **Extract motivation:**
   - Read commit bodies for context
   - Look for issue references
   - Understand the "why"

6. **Determine testing needs:**
   - Check for new test files
   - Look for test updates
   - Identify manual testing requirements

7. **Identify breaking changes:**
   - Look for "BREAKING CHANGE" in commits
   - Check for API changes
   - Look for removed features

## Generation Examples

### Feature Addition

**Given commits:**
```
feat(auth): add OAuth2 authentication
feat(auth): add Google OAuth provider
test(auth): add OAuth integration tests
docs(auth): update authentication guide
```

**Generated PR description:**
```markdown
## Description

Add OAuth2 authentication support, allowing users to sign in with Google accounts.

## Type of Change

- [x] New feature (non-breaking change adding functionality)

## What Changed

- Implemented OAuth2 authorization code flow
- Added Google OAuth provider configuration
- Created callback endpoint for token exchange
- Updated user model to store OAuth provider info
- Added integration tests for OAuth flow
- Updated authentication documentation

## Why

Users requested the ability to sign in with existing Google accounts instead of creating new passwords. This reduces friction during signup and improves security by delegating authentication to trusted providers.

Closes #45

## Testing

- [x] Unit tests added/updated
- [x] Integration tests added/updated
- [x] Tested manually

**Manual testing steps:**
1. Click "Sign in with Google" button
2. Authorize app in Google OAuth screen
3. Verify redirect back to app with user logged in
4. Check user profile shows Google as auth provider

**Test coverage:** 87% (↑ 3% from baseline)

## Deployment Notes

- [x] Requires environment variable changes

**Instructions:**
Add these environment variables:
- `GOOGLE_CLIENT_ID` - OAuth client ID from Google Console
- `GOOGLE_CLIENT_SECRET` - OAuth client secret
- `OAUTH_CALLBACK_URL` - Callback URL (https://yourdomain.com/auth/callback)

## Checklist

- [x] Code follows project conventions
- [x] Self-reviewed the code
- [x] Added/updated documentation
- [x] Added/updated tests
- [x] All tests passing
- [x] No new linter warnings

## Related Issues/PRs

Closes #45

---

**Estimated review time:** 20 minutes
```

### Bug Fix

**Given commits:**
```
fix(api): resolve database connection timeout
test(api): add connection pool tests
```

**Generated PR description:**
```markdown
## Description

Fix database connection timeout errors occurring under load.

## Type of Change

- [x] Bug fix (non-breaking change fixing an issue)

## What Changed

- Increased connection pool size from 10 to 50
- Added connection recycling after 1 hour
- Added connection health checks
- Added tests for connection pool behavior

## Why

Users were experiencing timeout errors during peak traffic. Investigation showed the connection pool was exhausting, causing requests to wait indefinitely for available connections.

Fixes #123

## Testing

- [x] Unit tests added/updated
- [x] Integration tests added/updated
- [x] Tested manually

**Manual testing steps:**
1. Simulate high load with 100 concurrent requests
2. Monitor connection pool metrics
3. Verify no timeout errors occur
4. Check connection recycling after 1 hour

**Test coverage:** 82% (no change)

## Performance Impact

- [x] Improves performance

**Benchmarks:**
- Before: Timeout errors after 50 concurrent users
- After: Handles 200+ concurrent users without errors

## Checklist

- [x] Code follows project conventions
- [x] Self-reviewed the code
- [x] Added/updated tests
- [x] All tests passing
- [x] No new linter warnings

## Related Issues/PRs

Fixes #123

---

**Estimated review time:** 10 minutes
```

### Breaking Change

**Given commits:**
```
feat(api): change timestamp format to ISO 8601

BREAKING CHANGE: User endpoint now returns ISO 8601
timestamps instead of Unix timestamps.
```

**Generated PR description:**
```markdown
## Description

Change API timestamp format from Unix timestamps to ISO 8601 strings for better readability and time zone support.

## Type of Change

- [x] Breaking change (fix or feature causing existing functionality to break)

## What Changed

- Updated all API responses to use ISO 8601 timestamps
- Added timezone information to timestamps
- Updated API documentation
- Added migration guide

## Why

ISO 8601 is more human-readable and includes timezone information, preventing timezone-related bugs. This is standard practice for modern APIs.

Closes #234

## Breaking Changes

- [x] Yes

**What breaks:**
API responses now return timestamps as strings in ISO 8601 format instead of Unix timestamps.

**Migration path:**
```javascript
// Old (Unix timestamp)
{ "createdAt": 1642425600 }

// New (ISO 8601 string)
{ "createdAt": "2022-01-17T12:00:00Z" }

// Client code update:
// Old: new Date(response.createdAt * 1000)
// New: new Date(response.createdAt)
```

**Timeline:**
- v2.0.0: New format (this PR)
- v2.1.0: Remove old format completely

## Deployment Notes

- [x] Requires configuration changes

**Instructions:**
Update API version to v2.0.0 in deployment config.
Client applications must update to handle new timestamp format before deploying.

## Checklist

- [x] Code follows project conventions
- [x] Self-reviewed the code
- [x] Added/updated documentation
- [x] Added/updated tests
- [x] All tests passing
- [x] Updated CHANGELOG

## Related Issues/PRs

Closes #234

## Reviewer Notes

- Focus review on timestamp serialization logic
- Verify all timestamp fields updated consistently
- Check migration guide clarity

---

**Estimated review time:** 25 minutes
```

## Instructions

1. **Analyze commits:**
   ```bash
   git log main..HEAD --format="%s%n%b"
   ```

2. **Get file changes:**
   ```bash
   git diff main...HEAD --name-only
   git diff main...HEAD --stat
   ```

3. **Determine PR type:**
   - Look at commit types (feat/fix/etc.)
   - Check for breaking changes
   - Identify primary purpose

4. **Extract key information:**
   - What changed (from diff)
   - Why it changed (from commit messages)
   - How it was tested (look for test files)
   - Breaking changes (from commits)
   - Related issues (from commit footers)

5. **Fill template:**
   - Check appropriate boxes
   - Fill in descriptions
   - Add testing details
   - Note deployment requirements

6. **Format output:**
   - Use markdown
   - Include checklists
   - Add relevant sections
   - Keep concise but complete

## Best Practices

✅ **DO:**
- Summarize changes concisely
- Explain the "why" not just "what"
- Include testing information
- Note breaking changes
- Reference related issues
- Provide migration guides for breaking changes
- Estimate review time
- Highlight areas needing careful review

❌ **DON'T:**
- Just list commit messages
- Skip the "why"
- Forget to mention breaking changes
- Omit testing details
- Leave sections empty (use "N/A" if not applicable)
- Make reviewers guess context

## Constraints

- Must use markdown format
- Must include all required sections
- Must check appropriate boxes
- Must reference issues if applicable
- Must note breaking changes if present
- Must include testing information
- Keep description under 500 words (be concise)
