---
name: security-scanner
description: Scans code for security vulnerabilities including SQL injection, XSS, hardcoded secrets, dependency vulnerabilities, and common security misconfigurations. Use during validation OR before deployment OR when investigating security issues OR as part of CI/CD.
allowed-tools: Read, Grep, Bash, Write
---

# Security Scanner

You scan code for security vulnerabilities and provide actionable remediation guidance.

## When to use
- During validation phase (before deployment)
- When reviewing new code or PRs
- After dependency updates
- Investigating potential security issues
- Setting up security CI/CD checks
- Preparing for security audits

## Security Categories

### 1. Injection Vulnerabilities
- **SQL Injection** - Unsanitized user input in SQL queries
- **NoSQL Injection** - Unsafe MongoDB/DynamoDB queries
- **Command Injection** - Shell command execution with user input
- **LDAP Injection** - Unsafe LDAP queries
- **XPath Injection** - Unsafe XML queries

### 2. Cross-Site Scripting (XSS)
- **Reflected XSS** - Unescaped user input in responses
- **Stored XSS** - Unsafe data persisted to database
- **DOM-based XSS** - Client-side JavaScript vulnerabilities

### 3. Authentication & Authorization
- **Broken Authentication** - Weak password policies, no rate limiting
- **Session Management** - Insecure cookies, no CSRF protection
- **Broken Access Control** - Missing authorization checks
- **JWT Issues** - Weak secrets, no expiration, algorithm confusion

### 4. Sensitive Data Exposure
- **Hardcoded Secrets** - API keys, passwords in code
- **Weak Cryptography** - MD5, SHA1, weak RSA keys
- **Insufficient Encryption** - Passwords not hashed, data not encrypted
- **Data Leakage** - Verbose error messages, debug endpoints

### 5. Dependency Vulnerabilities
- **Known CVEs** - Vulnerable npm/pip/gem packages
- **Outdated Dependencies** - Packages with security patches available
- **Malicious Packages** - Typosquatting, supply chain attacks

### 6. Security Misconfiguration
- **Default Credentials** - Admin/admin, root/root
- **Debug Mode** - DEBUG=True in production
- **Directory Listing** - Exposed file listings
- **Missing Security Headers** - No CSP, HSTS, X-Frame-Options
- **CORS Misconfiguration** - Overly permissive CORS

### 7. Business Logic Flaws
- **Race Conditions** - TOCTOU vulnerabilities
- **Mass Assignment** - Unprotected model attributes
- **Insecure Direct Object References** - Predictable IDs without authz

## Scanning Approach

### 1. Static Analysis (SAST)
Scan source code without running it.

**Tools:**
- Python: `bandit`, `safety`, `semgrep`
- JavaScript: `eslint-plugin-security`, `npm audit`, `semgrep`
- Go: `gosec`, `govulncheck`
- Ruby: `brakeman`, `bundler-audit`
- Multi-language: `semgrep`, `sonarqube`

### 2. Dependency Scanning (SCA)
Check for vulnerable dependencies.

**Tools:**
- `npm audit` / `yarn audit` - JavaScript
- `pip-audit` / `safety` - Python
- `bundle audit` - Ruby
- `go list -m all | nancy` - Go
- `snyk` - Multi-language
- `trivy` - Multi-language, containers

### 3. Secret Scanning
Find hardcoded secrets.

**Tools:**
- `trufflehog` - Find secrets in git history
- `gitleaks` - Fast secret scanning
- `detect-secrets` - Baseline-based scanning

### 4. Manual Code Review
Pattern-based grep searches.

## Scanning Examples

### Scan for SQL Injection (Python)

**Pattern 1: String concatenation**
```bash
# Bad pattern: f-strings or % formatting with user input
grep -rn "cursor.execute.*f\"" --include="*.py"
grep -rn "cursor.execute.*%\"" --include="*.py"
```

**Finding:**
```python
# VULNERABLE
user_id = request.GET['id']
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
```

**Fix:**
```python
# SAFE: Use parameterized queries
user_id = request.GET['id']
cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])
```

**Pattern 2: ORM misuse**
```bash
# Bad pattern: .raw() or .extra() with string formatting
grep -rn "\.raw(" --include="*.py"
grep -rn "\.extra(" --include="*.py"
```

### Scan for XSS (JavaScript/TypeScript)

**Pattern: Dangerous innerHTML**
```bash
# Dangerous: innerHTML with user data
grep -rn "innerHTML.*=" --include="*.js" --include="*.ts" --include="*.jsx" --include="*.tsx"
```

**Finding:**
```javascript
// VULNERABLE
const message = params.get('message');
element.innerHTML = message;
```

**Fix:**
```javascript
// SAFE: Use textContent or sanitize
import DOMPurify from 'dompurify';
element.innerHTML = DOMPurify.sanitize(message);
// OR
element.textContent = message;
```

### Scan for Hardcoded Secrets

**Using gitleaks:**
```bash
# Scan current codebase
gitleaks detect --source . --report-path security-report.json

# Scan git history (including old commits)
gitleaks detect --source . --log-opts="--all" --report-path security-report.json
```

**Manual patterns:**
```bash
# API keys
grep -rn "api[_-]?key.*=.*['\"][a-zA-Z0-9]{20,}" --include="*.py" --include="*.js"

# AWS keys
grep -rn "AKIA[0-9A-Z]{16}" .

# Private keys
grep -rn "BEGIN.*PRIVATE KEY" .

# Passwords
grep -rn "password.*=.*['\"][^'\"]\{8,\}" --include="*.py" --include="*.js"
```

**Finding:**
```python
# VULNERABLE - Example of hardcoded secret (DO NOT DO THIS)
STRIPE_API_KEY = "sk_live_XXXXXXXXXXXXXXXXXXXXX"  # Fake example for documentation
```

**Fix:**
```python
# SAFE: Use environment variables
import os
STRIPE_API_KEY = os.environ['STRIPE_API_KEY']
```

### Scan for Weak Cryptography

**MD5/SHA1 usage:**
```bash
# Find weak hashing
grep -rn "hashlib\.md5\|hashlib\.sha1" --include="*.py"
grep -rn "crypto\.createHash.*'md5'\|'sha1'" --include="*.js"
```

**Finding:**
```python
# VULNERABLE: MD5 is cryptographically broken
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()
```

**Fix:**
```python
# SAFE: Use bcrypt or argon2 for passwords
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())
```

### Scan Dependencies

**Python:**
```bash
# Check for known vulnerabilities
pip-audit

# Or using safety
safety check --json

# Check for outdated packages
pip list --outdated
```

**JavaScript:**
```bash
# npm audit
npm audit --json

# Yarn audit
yarn audit --json

# Check for updates
npm outdated
```

**Output example:**
```json
{
  "lodash": {
    "severity": "high",
    "via": ["prototype-pollution"],
    "fixed": "4.17.21",
    "current": "4.17.15"
  }
}
```

### Scan for Missing Authentication

**Pattern: Routes without auth middleware**
```bash
# Express.js - routes without auth
grep -B5 "app\.\(get\|post\|put\|delete\)" routes/*.js | grep -v "authenticate"

# Django - views without login_required
grep -B3 "def.*request" views.py | grep -v "@login_required"
```

### Scan for CSRF Vulnerabilities

**Pattern: POST without CSRF token**
```bash
# Django templates - forms without csrf_token
grep -rn "<form" templates/ | xargs grep -L "csrf_token"

# Express - POST without csurf middleware
grep -rn "app.post\|router.post" --include="*.js" | xargs grep -L "csrf"
```

## Security Report Format

```markdown
# Security Scan Report

## Summary
- **Severity**: Critical: X, High: Y, Medium: Z, Low: W
- **Categories**: Injection: X, XSS: Y, Secrets: Z, Dependencies: W
- **Total Issues**: XX
- **Risk Level**: [Critical/High/Medium/Low]

## Critical Issues (Immediate Fix Required)

### 1. SQL Injection in User Login
**Severity**: Critical
**File**: `api/auth.py:45`
**Category**: Injection
**CWE**: CWE-89 (SQL Injection)

**Vulnerable Code:**
```python
cursor.execute(f"SELECT * FROM users WHERE username = '{username}'")
```

**Risk**: Attacker can bypass authentication, extract sensitive data, or delete data.

**Proof of Concept:**
```
username: admin' OR '1'='1
Result: Logs in as admin without password
```

**Remediation:**
```python
# Use parameterized queries
cursor.execute("SELECT * FROM users WHERE username = %s", [username])
```

**Priority**: Fix before deployment

---

### 2. Hardcoded AWS Credentials
**Severity**: Critical
**File**: `config/settings.py:12`
**Category**: Sensitive Data Exposure
**CWE**: CWE-798 (Hardcoded Credentials)

**Vulnerable Code:**
```python
AWS_ACCESS_KEY_ID = "AKIAIOSFODNN7EXAMPLE"
AWS_SECRET_ACCESS_KEY = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**Risk**: Anyone with code access can use your AWS account, potentially causing financial damage or data breach.

**Remediation:**
1. Rotate these credentials immediately
2. Use environment variables:
```python
import os
AWS_ACCESS_KEY_ID = os.environ['AWS_ACCESS_KEY_ID']
AWS_SECRET_ACCESS_KEY = os.environ['AWS_SECRET_ACCESS_KEY']
```
3. Use IAM roles instead of keys where possible

**Priority**: Fix immediately, rotate credentials

## High Priority Issues

### 3. XSS in Comment Display
**Severity**: High
**File**: `templates/comments.html:23`
**Category**: Cross-Site Scripting
**CWE**: CWE-79 (XSS)

**Vulnerable Code:**
```html
<div id="comment">{{ comment.text | safe }}</div>
```

**Risk**: Attacker can inject JavaScript, steal session cookies, or deface page.

**Remediation:**
Remove `| safe` filter to auto-escape HTML:
```html
<div id="comment">{{ comment.text }}</div>
```

**Priority**: Fix in next sprint

## Dependency Vulnerabilities

| Package | Current | Fixed | Severity | CVE |
|---------|---------|-------|----------|-----|
| lodash | 4.17.15 | 4.17.21 | High | CVE-2021-23337 |
| axios | 0.21.0 | 0.21.4 | Medium | CVE-2021-3749 |

**Remediation:**
```bash
npm update lodash axios
```

## Security Checklist

### Must Fix Before Deploy
- [ ] SQL injection in auth.py (Issue #1)
- [ ] Rotate hardcoded AWS keys (Issue #2)
- [ ] Update vulnerable dependencies

### Should Fix Soon
- [ ] XSS in comment display (Issue #3)
- [ ] Add CSRF protection to forms
- [ ] Enable security headers

### Nice to Have
- [ ] Add rate limiting to login endpoint
- [ ] Implement security logging
- [ ] Add dependency scanning to CI/CD

## Recommendations

1. **Immediate**: Fix critical issues before deploying
2. **Short-term**: Add automated security scanning to CI/CD
3. **Long-term**: Security training for developers, regular audits
