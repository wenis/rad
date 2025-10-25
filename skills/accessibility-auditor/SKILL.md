---
name: accessibility-auditor
description: Audits applications for WCAG 2.1 compliance including semantic HTML, keyboard navigation, ARIA attributes, color contrast, and screen reader support. Use during validation OR before deployment OR when adding new features OR after UI changes.
allowed-tools: Read, Grep, Bash, Write
---

# Accessibility Auditor

You audit applications for accessibility issues and provide specific remediation guidance to achieve WCAG 2.1 AA compliance.

## When to use
- Before deploying to production
- During code review for UI changes
- After adding new components or features
- When preparing for accessibility audits
- Investigating accessibility complaints
- Setting up accessibility CI/CD checks

## WCAG 2.1 Levels

### Level A (Minimum)
Basic accessibility features that must be present.

### Level AA (Target)
**Most common legal requirement**. Addresses major barriers.

### Level AAA (Enhanced)
Highest level, not always achievable for all content.

**This skill targets WCAG 2.1 Level AA compliance.**

## Audit Categories

### 1. Perceivable
Users must be able to perceive the information.

**Issues to check:**
- Missing alt text on images
- Poor color contrast
- Missing captions for videos
- Content only available through color

### 2. Operable
Users must be able to operate the interface.

**Issues to check:**
- Keyboard navigation broken
- Focus order incorrect
- Missing skip links
- Keyboard traps

### 3. Understandable
Users must be able to understand the interface.

**Issues to check:**
- Confusing labels
- Missing form validation messages
- Unexpected behavior
- Complex language without alternatives

### 4. Robust
Content must work with assistive technologies.

**Issues to check:**
- Invalid HTML
- Missing ARIA labels
- Incorrect ARIA roles
- Broken screen reader announcements

## Automated Checks

### Using axe-core (JavaScript)

**Install:**
```bash
npm install -D @axe-core/cli
```

**Run audit:**
```bash
# Audit a URL
npx axe http://localhost:3000 --exit

# Generate report
npx axe http://localhost:3000 --save audit-report.json

# Audit with specific rules
npx axe http://localhost:3000 --rules color-contrast,aria-roles
```

**Programmatic usage:**
```typescript
// tests/accessibility.test.ts
import { test, expect } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

test('should not have accessibility violations', async ({ page }) => {
  await page.goto('http://localhost:3000');

  const accessibilityScanResults = await new AxeBuilder({ page }).analyze();

  expect(accessibilityScanResults.violations).toEqual([]);
});
```

### Using Pa11y

**Install:**
```bash
npm install -g pa11y
```

**Run:**
```bash
# Audit a page
pa11y http://localhost:3000

# Generate JSON report
pa11y http://localhost:3000 --reporter json > audit.json

# Audit with specific standard
pa11y http://localhost:3000 --standard WCAG2AA
```

### Using Lighthouse

**Run via CLI:**
```bash
lighthouse http://localhost:3000 --only-categories=accessibility --output=json --output-path=./audit.json
```

**In Chrome DevTools:**
1. Open DevTools (F12)
2. Go to Lighthouse tab
3. Check "Accessibility"
4. Click "Generate report"

## Manual Checks

Automated tools catch ~30-40% of issues. Manual testing is critical.

### Keyboard Navigation

**Test checklist:**
- [ ] Tab through entire page in logical order
- [ ] All interactive elements reachable
- [ ] Focus indicator visible on all elements
- [ ] Enter/Space activates buttons/links
- [ ] Escape closes modals/dropdowns
- [ ] Arrow keys navigate within components (tabs, menus)
- [ ] No keyboard traps (can Tab out of everything)
- [ ] Skip links work (skip to main content)

**Common issues:**
```html
<!-- BAD: div used as button -->
<div onclick="handleClick()">Click me</div>

<!-- GOOD: semantic button -->
<button onclick="handleClick()">Click me</button>

<!-- BAD: Missing tabindex for custom interactive element -->
<div class="custom-button">Click</div>

<!-- GOOD: Tabindex and role for custom element -->
<div role="button" tabindex="0" class="custom-button">Click</div>
```

### Screen Reader Testing

**Tools:**
- **VoiceOver** (Mac): Cmd+F5
- **NVDA** (Windows): Free, recommended
- **JAWS** (Windows): Industry standard, paid

**Test checklist:**
- [ ] Page title announced
- [ ] Headings read in order (h1, h2, h3...)
- [ ] Images have descriptive alt text
- [ ] Forms have associated labels
- [ ] Error messages announced
- [ ] Dynamic content changes announced
- [ ] Links have meaningful text (not "click here")

**Common issues:**
```html
<!-- BAD: No label for input -->
<input type="text" placeholder="Name" />

<!-- GOOD: Explicit label -->
<label for="name">Name</label>
<input id="name" type="text" />

<!-- BAD: Generic link text -->
<a href="/learn-more">Click here</a>

<!-- GOOD: Descriptive link text -->
<a href="/learn-more">Learn more about our pricing</a>

<!-- BAD: Image without alt text -->
<img src="logo.png" />

<!-- GOOD: Image with alt text -->
<img src="logo.png" alt="Company logo" />

<!-- GOOD: Decorative image (empty alt) -->
<img src="decorative-pattern.png" alt="" />
```

### Color Contrast

**WCAG AA Requirements:**
- Normal text: 4.5:1 minimum
- Large text (18pt+ or 14pt+ bold): 3:1 minimum
- UI components: 3:1 minimum

**Tools:**
```bash
# Use axe or Pa11y for automated checks
pa11y http://localhost:3000 --standard WCAG2AA --reporter json | grep "color-contrast"

# Manual check with browser extension
# Install: WCAG Color Contrast Checker
```

**Common issues:**
```css
/* BAD: Insufficient contrast */
.text {
  color: #767676; /* Gray on white = 4.4:1 */
  background: #ffffff;
}

/* GOOD: Sufficient contrast */
.text {
  color: #595959; /* Gray on white = 7:1 */
  background: #ffffff;
}

/* BAD: Color-only indication */
.error {
  color: red;
}

/* GOOD: Color + icon/text */
.error {
  color: red;
}
.error::before {
  content: "⚠️ Error: ";
}
```

### Focus Management

**Test checklist:**
- [ ] Focus indicator visible (outline or custom style)
- [ ] Focus trapped in modals
- [ ] Focus restored after modal close
- [ ] Focus moves to new content after dynamic updates

**Common issues:**
```css
/* BAD: Removing focus outline entirely */
*:focus {
  outline: none;
}

/* GOOD: Custom focus style */
*:focus-visible {
  outline: 2px solid #2563eb;
  outline-offset: 2px;
}
```

```typescript
// BAD: No focus management in modal
function Modal({ isOpen, children }) {
  if (!isOpen) return null;
  return <div>{children}</div>;
}

// GOOD: Focus trap in modal
import { useRef, useEffect } from 'react';
import FocusTrap from 'focus-trap-react';

function Modal({ isOpen, onClose, children }) {
  const closeButtonRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) {
      closeButtonRef.current?.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <FocusTrap>
      <div role="dialog" aria-modal="true">
        <button ref={closeButtonRef} onClick={onClose}>
          Close
        </button>
        {children}
      </div>
    </FocusTrap>
  );
}
```

### ARIA Usage

**Rules:**
1. Use semantic HTML first (ARIA should be rare)
2. No ARIA is better than bad ARIA
3. ARIA doesn't change behavior, only how screen readers announce

**Common patterns:**
```html
<!-- Form with live validation -->
<form>
  <label for="email">Email</label>
  <input
    id="email"
    type="email"
    aria-describedby="email-error"
    aria-invalid="true"
  />
  <p id="email-error" role="alert">
    Please enter a valid email address
  </p>
</form>

<!-- Loading state -->
<button aria-busy="true" aria-live="polite">
  Loading...
</button>

<!-- Tab component -->
<div role="tablist">
  <button
    role="tab"
    aria-selected="true"
    aria-controls="panel-1"
    id="tab-1"
  >
    Tab 1
  </button>
  <button
    role="tab"
    aria-selected="false"
    aria-controls="panel-2"
    id="tab-2"
  >
    Tab 2
  </button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">
  Content 1
</div>
```

## Audit Report Format

```markdown
# Accessibility Audit Report

## Summary
- **Date**: 2025-01-24
- **URL/Pages Audited**: http://localhost:3000/
- **Tools Used**: axe-core, Pa11y, manual testing
- **WCAG Level**: AA
- **Total Issues**: 23 (8 critical, 10 serious, 5 moderate)

## Critical Issues (Must Fix)

### 1. Missing Form Labels
**WCAG**: 3.3.2 Labels or Instructions (Level A)
**Impact**: High - Users cannot identify form fields
**Pages**: Contact form, Login page

**Issue:**
```html
<input type="email" placeholder="Email" />
```

**Fix:**
```html
<label for="email">Email</label>
<input id="email" type="email" />
```

**Files to update:**
- `src/components/ContactForm.tsx:45`
- `src/pages/login.tsx:23`

---

### 2. Insufficient Color Contrast
**WCAG**: 1.4.3 Contrast (Level AA)
**Impact**: High - Text unreadable for low vision users
**Contrast Ratio**: 3.2:1 (requires 4.5:1)

**Issue:**
```css
.subtitle {
  color: #999999; /* Gray on white */
}
```

**Fix:**
```css
.subtitle {
  color: #595959; /* Darker gray = 7:1 contrast */
}
```

**Files to update:**
- `src/styles/typography.css:34`
- `src/components/Hero.tsx:12` (inline style)

---

### 3. Keyboard Navigation Broken
**WCAG**: 2.1.1 Keyboard (Level A)
**Impact**: Critical - Cannot use site with keyboard only

**Issue:**
Custom dropdown not keyboard accessible.

**Fix:**
```typescript
// Add keyboard handlers
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  Open dropdown
</div>
```

**Files to update:**
- `src/components/Dropdown.tsx:67`

## Serious Issues (Should Fix)

### 4. Images Without Alt Text
**WCAG**: 1.1.1 Non-text Content (Level A)
**Count**: 12 images
**Impact**: Screen readers cannot describe images

**Files:**
- `src/components/Gallery.tsx` - 8 images
- `src/pages/about.tsx` - 4 images

**Fix:**
Add descriptive alt text:
```html
<img src="team.jpg" alt="Our team at the 2024 conference" />
```

---

### 5. Missing Page Title
**WCAG**: 2.4.2 Page Titled (Level A)
**Pages**: Dashboard, Settings
**Impact**: Users cannot identify page in browser tabs

**Fix:**
```typescript
// Next.js
import Head from 'next/head';

export default function Dashboard() {
  return (
    <>
      <Head>
        <title>Dashboard - MyApp</title>
      </Head>
      {/* Page content */}
    </>
  );
}
```

## Moderate Issues (Nice to Fix)

### 6. Inconsistent Heading Hierarchy
**WCAG**: 1.3.1 Info and Relationships (Level A)
**Impact**: Moderate - Screen reader navigation harder

**Issue:**
Skipping from h1 to h3 (missing h2).

**Fix:**
Use proper heading hierarchy (h1 → h2 → h3).

## Testing Checklist

### Automated Tests
- [x] axe-core: 15 issues found
- [x] Pa11y: 18 issues found
- [x] Lighthouse: Score 67/100

### Manual Tests
- [x] Keyboard navigation (8 issues)
- [x] Screen reader (VoiceOver) (5 issues)
- [x] Color contrast (3 issues)
- [ ] Zoom to 200% (pending)
- [ ] High contrast mode (pending)

## Recommendations by Priority

### Immediate (Before Launch)
1. Add labels to all form inputs
2. Fix color contrast issues
3. Implement keyboard navigation for custom components
4. Add alt text to all images
5. Add page titles

### Short-term (Next Sprint)
1. Fix heading hierarchy
2. Add skip links
3. Implement focus management in modals
4. Add ARIA labels where needed
5. Test with multiple screen readers

### Long-term (Continuous)
1. Set up automated accessibility tests in CI
2. Add accessibility to design review process
3. Train team on accessibility best practices
4. Conduct regular audits (quarterly)
5. Add accessibility section to component documentation

## Testing Setup for CI/CD

```yaml
# .github/workflows/accessibility.yml
name: Accessibility Tests

on: [push, pull_request]

jobs:
  a11y:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Start server
        run: npm start &

      - name: Wait for server
        run: npx wait-on http://localhost:3000

      - name: Run axe tests
        run: npx @axe-core/cli http://localhost:3000 --exit

      - name: Run Pa11y
        run: npx pa11y-ci --sitemap http://localhost:3000/sitemap.xml
```

## Resources
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)
- [axe DevTools](https://www.deque.com/axe/devtools/)
- [WebAIM](https://webaim.org/)
- [A11y Project Checklist](https://www.a11yproject.com/checklist/)
```

## Instructions

1. **Run automated tools**: axe, Pa11y, Lighthouse
2. **Parse results**: Extract violations by severity
3. **Manual testing**:
   - Keyboard navigation
   - Screen reader testing
   - Color contrast checks
   - Focus management
4. **Categorize issues**: Critical, serious, moderate
5. **Provide fixes**: Specific code changes for each issue
6. **Generate report**: Structured markdown with priorities
7. **Create test setup**: CI/CD configuration for automation

## Best Practices

✅ **DO:**
- Test with real assistive technologies
- Involve users with disabilities in testing
- Fix issues at the source (design/component library)
- Automate what you can
- Make accessibility part of definition of done
- Test on real devices (mobile screen readers)

❌ **DON'T:**
- Rely only on automated tools (catch ~30-40%)
- Use ARIA when semantic HTML works
- Remove focus indicators without replacement
- Assume "accessible" means "ugly"
- Skip keyboard testing
- Ignore color-blind users

## Constraints

- Must test with keyboard only (no mouse)
- Must test with at least one screen reader
- Must check color contrast programmatically
- Must validate HTML before ARIA audit
- Report must include specific code fixes
- Must prioritize by user impact, not tool severity
