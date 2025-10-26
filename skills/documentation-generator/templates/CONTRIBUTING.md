# Contributing Guidelines Template

Use this template to create contribution guidelines for your project.

```markdown
# Contributing to [Project Name]

Thank you for your interest in contributing! This document provides guidelines for contributing to this project.

## Code of Conduct

Be respectful, inclusive, and constructive. See [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

## Getting Started

1. **Fork the repository**
2. **Clone your fork**: `git clone https://github.com/your-username/project.git`
3. **Create a branch**: `git checkout -b feature/my-feature`
4. **Make changes** (see Development Workflow below)
5. **Test thoroughly**
6. **Submit pull request**

## Development Workflow

### 1. Set Up Environment

\`\`\`bash
# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Start services
docker-compose up -d

# Run migrations
npm run migrate
\`\`\`

### 2. Make Changes

- **Follow style guide** (Prettier, ESLint)
- **Write tests** for new features
- **Update documentation** if needed
- **Commit frequently** with clear messages

### 3. Test Your Changes

\`\`\`bash
# Run tests
npm test

# Run linter
npm run lint

# Type check
npm run typecheck

# Check coverage
npm run test:coverage
\`\`\`

### 4. Commit Guidelines

Follow [Conventional Commits](https://www.conventionalcommits.org/):

\`\`\`
feat: add user authentication
fix: resolve database connection issue
docs: update README with new setup steps
chore: upgrade dependencies
refactor: simplify payment processing
test: add integration tests for API
\`\`\`

### 5. Pull Request Process

**Before submitting**:
- [ ] Tests pass
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] No merge conflicts
- [ ] Linked to issue (if applicable)

**PR Template**:
\`\`\`markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
How was this tested?

## Checklist
- [ ] Tests pass
- [ ] Code reviewed
- [ ] Documentation updated
\`\`\`

## Code Standards

### Style Guide

- **TypeScript**: Follow ESLint rules, use Prettier
- **Python**: Follow PEP 8, use Black formatter
- **Naming**: camelCase (JS), snake_case (Python)

### Testing Standards

- **Unit tests**: Test individual functions
- **Integration tests**: Test API endpoints
- **Coverage**: Minimum 80% for new code

### Documentation

- **Functions**: JSDoc or docstrings
- **Complex logic**: Inline comments
- **API changes**: Update OpenAPI spec

## Review Process

1. **Automated checks**: CI/CD runs tests, linting
2. **Code review**: At least one approval required
3. **QA testing**: For significant features
4. **Merge**: Squash and merge to main

## Questions?

- Open an issue
- Ask in #engineering on Slack
- Email: dev@example.com
```
