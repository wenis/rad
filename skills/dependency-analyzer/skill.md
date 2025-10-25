---
name: dependency-analyzer
description: Analyzes code dependencies to identify which modules can be built in parallel vs sequentially. Use when planner creates build plans OR when determining module boundaries OR when optimizing parallel execution strategy OR when assessing feature complexity.
allowed-tools: Read, Grep, Glob, Bash
---

# Dependency Analyzer

You analyze code structure and dependencies to identify optimal module boundaries for parallel development.

## When to use
- Planner agent creating build plans with parallel strategy
- Determining if a feature can be parallelized effectively
- Analyzing existing codebase structure before planning
- Identifying module boundaries for a new feature
- Validating that proposed modules don't have hidden dependencies

## Analysis types

### 1. Feature Dependency Analysis
Analyze a feature specification to suggest module breakdown.

**Input**: Feature spec describing what needs to be built
**Output**: Recommended module structure with dependency graph

### 2. Codebase Structure Analysis
Analyze existing code to understand current module boundaries.

**Input**: Directory or file patterns to analyze
**Output**: Dependency graph showing relationships

### 3. Module Isolation Verification
Check if proposed modules are truly independent.

**Input**: List of proposed modules with file paths
**Output**: Validation report showing any hidden dependencies

## Analysis process

1. **Identify code components**: Find files, functions, classes related to the feature
2. **Extract imports/dependencies**: Parse import statements, function calls, shared state
3. **Build dependency graph**: Map what depends on what
4. **Detect cycles**: Find circular dependencies (blocks parallelization)
5. **Calculate dependency depth**: Identify phases (depth 0 = no deps, depth 1 = depends on depth 0, etc.)
6. **Group independent modules**: Modules with same depth can be parallel
7. **Generate recommendations**: Suggest optimal phase breakdown

## Dependency detection patterns

### Python
```bash
# Find imports
grep -rn "^import\|^from" --include="*.py" <path>

# Find function calls between modules
grep -rn "from .* import\|import .*\.<module>" --include="*.py"

# Find global state usage
grep -rn "global\|globals()" --include="*.py"
```

### JavaScript/TypeScript
```bash
# Find imports
grep -rn "^import\|require(" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx"

# Find dynamic imports
grep -rn "import(" --include="*.js" --include="*.ts"

# Find shared state
grep -rn "export.*let\|export.*var" --include="*.js" --include="*.ts"
```

### Go
```bash
# Find imports
grep -rn "^import" --include="*.go"

# Find package dependencies
go list -m all
```

## Output format

```markdown
# Dependency Analysis Report

## Feature: [Feature Name]

## Recommended Module Structure

### Complexity Assessment
**Overall Complexity**: [Simple/Moderate/Complex]
**Parallelization Potential**: [High/Medium/Low]
**Recommended Strategy**: [Parallel: N modules in M phases / Sequential: Single builder]

**Rationale**: [Why this strategy is optimal]

## Dependency Graph

```
Phase 1 (No dependencies - Can build in parallel):
  - Module A: auth/password.py
  - Module B: auth/jwt.py
  - Module C: services/email.py

Phase 2 (Depends on Phase 1):
  - Module D: api/auth.py (depends on A, B)
  - Module E: api/reset.py (depends on A, C)

Integration (Depends on all phases):
  - Wire modules together
  - Create integration layer
```

## Detailed Module Analysis

### Module A: Password Validation
**Files**:
- auth/password.py (new)
- tests/test_password.py (new)

**Dependencies**: None (pure function, no external deps)

**Exports**:
- `validate_password(password: str) -> ValidationResult`
- `hash_password(password: str) -> str`
- `verify_password(password: str, hash: str) -> bool`

**Used By**:
- Module D (api/auth.py) - for login validation
- Module E (api/reset.py) - for password reset

**Estimated Complexity**: Low - Standard password hashing library

**Parallel Build**: ✅ Can build independently

---

### Module B: JWT Token Management
**Files**:
- auth/jwt.py (new)
- tests/test_jwt.py (new)

**Dependencies**: None (uses standard JWT library)

**Exports**:
- `generate_token(user_id: str) -> str`
- `verify_token(token: str) -> dict`
- `refresh_token(token: str) -> str`

**Used By**:
- Module D (api/auth.py) - for token generation on login

**Estimated Complexity**: Low - Standard JWT operations

**Parallel Build**: ✅ Can build independently

---

## Integration Points

**A ↔ D**: Module D calls Module A functions
- Interface: `validate_password()`, `hash_password()`, `verify_password()`
- Integration type: Function calls
- Risk: Low - clear interface

**B ↔ D**: Module D calls Module B functions
- Interface: `generate_token()`, `verify_token()`
- Integration type: Function calls
- Risk: Low - standard JWT flow

**A ↔ E**: Module E calls Module A functions
- Interface: `hash_password()`, `verify_password()`
- Integration type: Function calls
- Risk: Low - password reset uses same validation

**C ↔ E**: Module E calls Module C functions
- Interface: `send_reset_email()`
- Integration type: Async function call
- Risk: Medium - need to handle email failures gracefully

## Shared State Analysis

**Database**:
- Module D, E both access `users` table
- Risk: Medium - need to ensure consistent schema
- Recommendation: Define User model interface upfront

**Configuration**:
- All modules use environment variables (SECRET_KEY, EMAIL_CONFIG)
- Risk: Low - standard env var pattern

**No global state detected**: ✅ Good for parallel development

## Potential Issues

### ⚠️ Warning: Circular Dependency Risk
- None detected ✅

### ⚠️ Warning: Tight Coupling
- Modules D and E both depend on shared User model
- Recommendation: Create User model first or define interface

### ⚠️ Warning: Shared Resources
- Database connection pool shared across modules
- Recommendation: Define database interface upfront

## Recommendations

### For Parallel Execution ✅
1. Build Modules A, B, C in parallel (Phase 1)
2. Define User model interface before Phase 2
3. Build Modules D, E in parallel after Phase 1 completes (Phase 2)
4. Integration phase wires everything together

### Optimization Opportunities
- Modules A and B are simple - could even combine into one
- Module C (email) could be stubbed initially to unblock Module E

### Risk Mitigation
- Define interfaces first: PasswordValidator, TokenManager, EmailService
- Use dependency injection for database connections
- Create integration tests for cross-module workflows

## Estimated Timeline

**Sequential build**: ~25 minutes
- Module A: 3 min
- Module B: 3 min
- Module C: 8 min
- Module D: 5 min
- Module E: 4 min
- Integration: 2 min

**Parallel build**: ~12 minutes
- Phase 1 (parallel): 8 min (limited by slowest: Module C)
- Phase 2 (parallel): 5 min (limited by slowest: Module D)
- Integration: 2 min

**Speedup**: 2.1x faster with parallel approach ✅
```

## Analysis techniques

### Static analysis
- Parse import statements
- Analyze function calls
- Identify shared data structures
- Detect global state

### AST parsing (for deep analysis)
Use Python `ast` module or similar:
```python
import ast

def find_imports(file_path):
    with open(file_path) as f:
        tree = ast.parse(f.read())

    imports = []
    for node in ast.walk(tree):
        if isinstance(node, ast.Import):
            for alias in node.names:
                imports.append(alias.name)
        elif isinstance(node, ast.ImportFrom):
            imports.append(node.module)

    return imports
```

### Visualization
Generate dependency graphs using mermaid syntax:
```markdown
```mermaid
graph TD
    A[Module A: Password] --> D[Module D: Auth API]
    B[Module B: JWT] --> D
    A --> E[Module E: Reset API]
    C[Module C: Email] --> E
    D --> F[Integration]
    E --> F
```
```

## Instructions

1. **Understand the feature**: Read the spec or user description
2. **Identify components**: List all code pieces needed
3. **Analyze dependencies**: Use grep/glob to find imports, calls, shared state
4. **Build dependency graph**: Map relationships
5. **Calculate phases**: Group independent modules
6. **Generate report**: Use standard format with recommendations
7. **Suggest interfaces**: Recommend contracts for clean integration

## Tools to use

- **Grep**: Find import statements, function calls, patterns
- **Glob**: Discover related files
- **Read**: Parse code to understand relationships
- **Bash**: Run static analysis tools (ast, go list, etc.)

## Best practices

- Prefer smaller, focused modules over large monolithic ones
- Aim for 2-5 modules per phase (not too many, not too few)
- Modules should have clear, single responsibilities
- Avoid circular dependencies (blocks parallelization)
- Consider build time: balance parallelism with overhead
- Simple features don't need complex breakdown (1-2 modules is fine)

## Output artifacts

- Dependency analysis report (Markdown)
- Dependency graph (Mermaid diagram)
- Module interface recommendations
- Risk assessment
- Build time estimates

## Constraints

- Focus on code dependencies, not business logic
- Don't suggest modules that are too granular (overhead)
- Detect circular dependencies and flag them
- Identify shared state that could cause race conditions
- Consider testability when suggesting module boundaries
