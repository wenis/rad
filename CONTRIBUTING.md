# Contributing to RAD System

Thank you for your interest in contributing to the Rapid Agile Development (RAD) System! This document provides guidelines for contributing.

## How to Contribute

### Reporting Issues

- **Check existing issues** first to avoid duplicates
- **Use issue templates** when creating new issues
- **Be specific** - Include error messages, steps to reproduce, expected vs actual behavior
- **Environment details** - Claude Code version, OS, project type

### Suggesting Features

- **Open a discussion** before implementing major features
- **Explain the use case** - Why is this needed? Who benefits?
- **Consider alternatives** - What other approaches were considered?
- **Check alignment** - Does this fit the RAD philosophy (speed + quality)?

### Pull Requests

**Before submitting:**
1. Test your changes thoroughly
2. Update relevant documentation
3. Follow the code style (see below)
4. Write a clear PR description

**PR Title Format:**
Use [Conventional Commits](https://www.conventionalcommits.org/):
```
feat: add mobile app builder skill
fix: resolve validator loop issue
docs: update README installation steps
refactor: simplify planner agent prompt
```

**PR Description Template:**
```markdown
## Description
Brief summary of changes and motivation.

## Type of Change
- [ ] Bug fix
- [ ] New feature (skill, agent enhancement)
- [ ] Documentation update
- [ ] Refactoring

## Testing
- [ ] Tested with `/rapid-dev` workflow
- [ ] Verified agents read updated prompts correctly
- [ ] Tested edge cases

## Checklist
- [ ] Documentation updated
- [ ] Follows existing patterns
- [ ] No breaking changes (or documented if unavoidable)
```

## Development Guidelines

### Adding a New Skill

1. **Create skill directory:**
   ```bash
   mkdir skills/my-skill-name
   ```

2. **Create SKILL.md** following this template:
   ```markdown
   ---
   name: my-skill-name
   description: Brief description of what this skill does and when to use it
   allowed-tools: Read, Write, Edit, Bash, Grep
   ---

   # Skill Name

   You [description of what the skill does].

   ## When to use
   - Use case 1
   - Use case 2

   ## Instructions
   1. Step 1
   2. Step 2

   ## Examples
   [Detailed examples with code]

   ## Best Practices
   ✅ DO: [recommendations]
   ❌ DON'T: [anti-patterns]

   ## Constraints
   - [Requirements and limitations]
   ```

3. **Test the skill:**
   - Invoke it manually to verify it works
   - Test integration with relevant agents
   - Verify documentation is clear

4. **Update documentation:**
   - Add skill to README.md skills list
   - Update skill count in documentation
   - Add usage examples if helpful

### Modifying Agents

**Be careful when modifying agent prompts** - they're the core of the workflow.

**Testing changes:**
1. Run `/rapid-dev` on a test feature
2. Verify agent reads correct context files
3. Check that output artifacts are created correctly
4. Ensure feedback loop still works
5. **🆕 Test parallel workflow** (for complex features)

**Key principles:**
- Agents must read `docs/SYSTEM.md` FIRST
- Agents should read `docs/CONVENTIONS.md` if it exists
- Maintain backwards compatibility where possible
- Document breaking changes clearly
- **🆕 Support both sequential and parallel execution modes**

### Working with Parallel Workflow (v2.0)

**New in v2.0:** RAD System supports parallel module development.

**If modifying planner agent:**
- Ensure build plans include dependency analysis
- Test with both simple (sequential) and complex (parallel) features
- Verify `dependency-analyzer` skill integration
- Check module interface generation

**If modifying builder agent:**
- Test orchestration mode (spawning sub-builders)
- Test module mode (building single module)
- Test integration mode (wiring modules)
- Verify progress dashboard updates
- Check parallel spawn with Task tool (multiple calls in one message)

**If modifying validator agent:**
- Test module-level validation (isolated)
- Test integration-level validation (cross-module)
- Test system-level validation (comprehensive)
- Verify report formats for each level

**Testing parallel workflows:**
```bash
# Test with a complex feature (should trigger parallel)
/plan "add user authentication with email verification and password reset"
# Check that build plan has phases and modules

/build  # Should spawn multiple builders
# Verify progress dashboard shows parallel execution

/validate  # Should show module-level and integration reports
```

**Key files to check:**
- `docs/specs/[feature].md` - Contains "Build Plan" section
- `docs/progress/[feature]-build-progress.md` - Real-time status
- `docs/validation/[feature]-[module]-report.md` - Module reports
- `docs/validation/[feature]-integration-report.md` - Integration report

### Modifying Commands

**Slash commands** orchestrate the workflow.

**Testing changes:**
1. Run the command manually
2. Verify it invokes correct agents
3. Check output is user-friendly
4. Ensure error handling works

### Documentation Standards

**README files:**
- Keep concise and scannable
- Include examples
- Update TOC if adding sections
- Use proper markdown formatting

**Agent/Skill prompts:**
- Clear "When to use" section
- Detailed instructions
- Concrete examples
- Best practices and constraints

**Code comments:**
- Explain *why*, not *what*
- Use markdown in .md files
- Keep formatting consistent

## Code Style

**Markdown files:**
- Use `#` for h1, `##` for h2, etc.
- Use `**bold**` and `*italic*` sparingly
- Code blocks with language hints: \`\`\`typescript
- Lists with `-` for unordered, `1.` for ordered
- Keep line length reasonable (< 120 chars)

**File naming:**
- Skill names: `kebab-case` (e.g., `tdd-code-generator`)
- Agent names: `lowercase` (e.g., `planner.md`)
- Commands: `kebab-case` (e.g., `init-project.md`)
- Templates: `PascalCase.template.md` (e.g., `SYSTEM.template.md`)

**Directory structure:**
```
agents/          # Agent prompts
skills/          # Skill directories with SKILL.md
commands/        # Slash command definitions
templates/       # Document templates
```

## Testing Your Changes

### Manual Testing Checklist

**Basic Workflow:**
- [ ] Run `/init-project` successfully
- [ ] Run `/rapid-dev` on a small feature (sequential)
- [ ] Verify all agents execute correctly
- [ ] Check artifacts are created in `docs/`
- [ ] Verify feedback loop works (builder-validator)
- [ ] Test with different tech stacks (Python, Node.js, etc.)

**🆕 Parallel Workflow (v2.0):**
- [ ] Run `/rapid-dev` on complex feature (parallel)
- [ ] Verify build plan shows phases and modules
- [ ] Check progress dashboard generates and updates
- [ ] Verify multiple builders spawn simultaneously
- [ ] Check module-level validation reports
- [ ] Verify integration validation runs after modules
- [ ] Test with module validation failures (iteration loop)
- [ ] Verify metrics analyzer generates performance report

### Edge Cases to Test

- Missing `docs/SYSTEM.md` (first time usage)
- Validation failures (loop iterations)
- Breaking changes in code
- Large projects with many files
- Projects without tests

## Areas We Need Help

**High Priority:**
- 🎯 Mobile app development skills (React Native, Flutter)
- 🌐 Additional language support (Rust, Ruby, Java)
- 📊 Analytics and metrics integration
- 🔄 CI/CD integration improvements
- 🧪 More comprehensive testing strategies
- **🆕 Parallel workflow optimization** (improve build plan strategies)
- **🆕 Module pattern templates** (reusable parallel module structures)

**Medium Priority:**
- 📚 More examples and tutorials (especially parallel workflow)
- 🎨 Better error messages
- 🚀 Performance optimizations
- 🔍 Better debugging tools
- **🆕 Parallel build visualizations** (better progress tracking)
- **🆕 Historical metrics dashboard** (track build performance over time)

**Ideas Welcome:**
- New skills for specific domains
- Improvements to existing agents
- Better documentation
- Integration with other tools
- **🆕 AI-powered build plan optimization** (ML-based module breakdown)
- **🆕 Cross-project learning** (share patterns across projects)

## Community

**Questions?**
- Open a GitHub Discussion for general questions
- Open an Issue for bugs or feature requests
- Tag maintainers if you need urgent help

**Code of Conduct:**
- Be respectful and inclusive
- Focus on constructive feedback
- Help newcomers learn
- Assume good intentions

## Release Process

We follow semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR:** Breaking changes to agent prompts or workflow
- **MINOR:** New skills, features, or enhancements
- **PATCH:** Bug fixes, documentation updates

**Maintainers:**
1. Review and merge PRs
2. Update CHANGELOG.md
3. Tag release
4. Update documentation

## Getting Help

**Stuck?**
1. Read `README.md` for workflow details
2. Check existing issues for similar problems
3. Open a discussion with specific questions
4. Tag `question` label on issues

**Want to pair?**
- Open an issue describing what you want to work on
- Maintainers can provide guidance
- We can discuss approach before you invest time

---

**Thank you for contributing!** Every improvement helps teams ship faster and more reliably.

Questions? Open an issue or discussion.
