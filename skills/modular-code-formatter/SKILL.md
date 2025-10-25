---
name: modular-code-formatter
description: Formats and refactors code for consistency, modularity, and best practices across Python, JavaScript, TypeScript, and Go. Use when code needs cleanup OR after rapid prototyping OR when enforcing code standards OR before code review.
allowed-tools: Read, Edit, Write, Bash
---

# Modular Code Formatter

You transform messy or prototype code into clean, production-ready code following industry best practices.

## When to use
- After rapid prototyping session (cleaning up "vibe code")
- Before code review or pull request
- When code has inconsistent style or structure
- When refactoring legacy code
- When enforcing team coding standards

## Core principles
1. **Modularity**: Break large functions into smaller, focused units
2. **Clarity**: Use descriptive names, remove ambiguity
3. **Consistency**: Follow language-specific style guides
4. **Simplicity**: Remove unnecessary complexity and duplication
5. **Documentation**: Add docstrings/comments only where needed

## Formatting rules by language

### Python (PEP 8)
- Max line length: 88 characters (Black standard)
- Imports: Standard lib → Third party → Local (alphabetized)
- Type hints for function signatures
- Docstrings for public functions/classes
- Snake_case for functions/variables, PascalCase for classes
- Use f-strings for formatting
- List comprehensions over map/filter when readable

### JavaScript/TypeScript
- Use `const` by default, `let` when reassignment needed
- Arrow functions for callbacks
- Destructuring for object/array access
- Template literals for string interpolation
- Async/await over raw promises
- Consistent semicolon usage (present or absent, not mixed)
- camelCase for variables/functions, PascalCase for classes
- Explicit types in TypeScript (no `any` without justification)

### Go
- Use `gofmt` formatting
- Error handling immediately after function calls
- Receiver names: short, consistent (e.g., `c` for `Client`)
- Package comments for exported identifiers
- Table-driven tests

## Refactoring patterns

### Extract function
**Before:**
```python
def process_order(order):
    # Validate
    if not order.get('id'):
        raise ValueError('Missing id')
    if not order.get('items'):
        raise ValueError('No items')

    # Calculate total
    total = 0
    for item in order['items']:
        total += item['price'] * item['quantity']

    # Apply discount
    if total > 100:
        total *= 0.9

    return total
```

**After:**
```python
def process_order(order: dict) -> float:
    """Process order and calculate final total with discounts."""
    validate_order(order)
    subtotal = calculate_subtotal(order['items'])
    return apply_discount(subtotal)


def validate_order(order: dict) -> None:
    """Validate order has required fields."""
    if not order.get('id'):
        raise ValueError('Missing order id')
    if not order.get('items'):
        raise ValueError('Order has no items')


def calculate_subtotal(items: list[dict]) -> float:
    """Calculate order subtotal from items."""
    return sum(item['price'] * item['quantity'] for item in items)


def apply_discount(amount: float) -> float:
    """Apply volume discount if applicable."""
    return amount * 0.9 if amount > 100 else amount
```

### Remove duplication
**Before:**
```typescript
function getUserEmail(userId: string): string {
  const response = fetch(`/api/users/${userId}`);
  const data = response.json();
  return data.email;
}

function getUserName(userId: string): string {
  const response = fetch(`/api/users/${userId}`);
  const data = response.json();
  return data.name;
}
```

**After:**
```typescript
async function getUser(userId: string): Promise<User> {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}

async function getUserEmail(userId: string): Promise<string> {
  const user = await getUser(userId);
  return user.email;
}

async function getUserName(userId: string): Promise<string> {
  const user = await getUser(userId);
  return user.name;
}
```

### Improve naming
**Before:**
```python
def fn(d):
    r = []
    for x in d:
        if x['t'] == 'a':
            r.append(x)
    return r
```

**After:**
```python
def get_active_users(users: list[dict]) -> list[dict]:
    """Filter and return only active users from the list."""
    return [user for user in users if user['type'] == 'active']
```

## Code organization

### File structure
```
module/
  __init__.py          # Public API
  core.py              # Core logic
  utils.py             # Helper functions
  models.py            # Data structures
  exceptions.py        # Custom exceptions
  tests/
    test_core.py
    test_utils.py
```

### Module organization
1. Docstring (module description)
2. Imports (grouped and sorted)
3. Constants
4. Exception classes
5. Public functions/classes
6. Private helpers (prefixed with _)
7. Main block (if executable)

## Instructions
1. **Read the code**: Understand what it does before changing it
2. **Run existing tests**: Ensure you don't break functionality
3. **Apply formatting**: Fix style issues, organize imports
4. **Refactor**: Extract functions, remove duplication, improve names
5. **Add documentation**: Docstrings for public API
6. **Run tests again**: Verify nothing broke
7. **Run linter**: Use language-specific tools (pylint, eslint, gofmt)

## Output format
For each file formatted:
1. List the improvements made
2. Show the reformatted code
3. Run linter/formatter to verify
4. Note any manual review needed

## Example workflow

**Input file (messy prototype):**
```python
def calc(x,y,op):
    if op=='add':
        return x+y
    elif op=='sub':
        return x-y
    elif op=='mul':
        return x*y
    elif op=='div':
        if y==0:
            return None
        return x/y
```

**Formatted output:**
```python
"""Calculator operations module."""

from typing import Optional


class DivisionByZeroError(Exception):
    """Raised when attempting to divide by zero."""


def calculate(x: float, y: float, operation: str) -> float:
    """
    Perform arithmetic operation on two numbers.

    Args:
        x: First operand
        y: Second operand
        operation: Operation to perform ('add', 'sub', 'mul', 'div')

    Returns:
        Result of the calculation

    Raises:
        DivisionByZeroError: If dividing by zero
        ValueError: If operation is not recognized
    """
    operations = {
        'add': lambda a, b: a + b,
        'sub': lambda a, b: a - b,
        'mul': lambda a, b: a * b,
        'div': _safe_divide,
    }

    if operation not in operations:
        raise ValueError(f'Unknown operation: {operation}')

    return operations[operation](x, y)


def _safe_divide(x: float, y: float) -> float:
    """Divide x by y, raising error if y is zero."""
    if y == 0:
        raise DivisionByZeroError('Cannot divide by zero')
    return x / y
```

**Improvements made:**
- Added type hints
- Proper error handling with custom exception
- Extracted division logic to private function
- Used dictionary dispatch instead of if/elif chain
- Added comprehensive docstrings
- Improved parameter names (op → operation)
- Consistent spacing and formatting

## Tools to run
After formatting, run:
- Python: `black .`, `isort .`, `mypy .`, `pylint .`
- JavaScript/TypeScript: `prettier --write .`, `eslint --fix .`
- Go: `gofmt -w .`, `go vet ./...`

## Constraints
- Do NOT change functionality or behavior
- Do NOT remove error handling
- Do NOT add unnecessary complexity
- Do NOT over-document obvious code
- Always verify tests still pass after formatting