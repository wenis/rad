# RAD System - Claude Code Configuration Repository

This repository contains the **Rapid Agile Development (RAD) System** - a complete workflow configuration for Claude Code.

## What This Repository Is

This is **not a project to build** - it's a **workflow system to install** into your projects.

**Purpose:** Provide AI agents (planner, builder, validator, shipper) that automate the software development lifecycle with quality gates and best practices.

## For Users (Installing RAD)

If you want to use RAD in your project:

```bash
cd your-project
git clone https://github.com/yourusername/rad .claude
/init-project "your project description"
/rapid-dev "feature to build"
```

See [README.md](README.md) for complete installation and usage instructions.

## For Contributors (Developing RAD)

If you want to improve the RAD system itself:

1. Clone this repository
2. Make changes to agents, skills, or commands
3. Test with `/rapid-dev` in a sample project
4. Submit PR following [CONTRIBUTING.md](CONTRIBUTING.md)

**Testing changes:**
```bash
# Clone RAD system
git clone https://github.com/yourusername/rad
cd rad

# Create test project
mkdir ../test-project
cd ../test-project

# Symlink RAD system for testing
ln -s ../rad .claude

# Test the workflow
/init-project "test project"
/rapid-dev "sample feature"
```

## Repository Structure

```
.claude/              # The RAD system (agents, skills, commands)
docs/                 # Documentation templates and examples
README.md             # Main documentation
CONTRIBUTING.md       # Contribution guidelines
LICENSE               # MIT License
```

## Notes for Claude Code

When working on this repository:
- This is a **workflow system**, not application code
- Changes should be tested with actual project workflows
- Agent prompts are the core - modify carefully
- Always update documentation when changing agents/skills
- Follow Conventional Commits for all changes
