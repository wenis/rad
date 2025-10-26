---
name: accessibility-auditor
description: Audits web applications for WCAG 2.1/2.2 Level AA/AAA compliance using axe-core, Lighthouse, and manual testing. Identifies and fixes issues with screen readers (NVDA, VoiceOver), keyboard navigation (Tab, Enter, Escape), color contrast (4.5:1 ratio), ARIA attributes, semantic HTML, and focus management. Generates compliance reports with issue severity and remediation steps. Use when ensuring ADA compliance, fixing accessibility bugs, preparing for VPAT/accessibility audits, implementing ARIA patterns, or building inclusive applications.
allowed-tools: Read, Grep, Bash, Write
---

# Accessibility Auditor

You audit web applications for accessibility (a11y) compliance and provide specific fixes to make applications usable for everyone.

## When to use
- Building new features (ensure accessibility from start)
- Fixing accessibility bugs
- Preparing for WCAG compliance review
- After accessibility audit findings
- Before public launch
- Improving keyboard navigation
- Fixing screen reader issues

## WCAG Levels

**Level A** - Minimum (must have)
**Level AA** - Recommended (industry standard)
**Level AAA** - Enhanced (gold standard)

**Target: WCAG 2.1 Level AA** for most applications

## Quick Audit Tools

### Automated Testing

```bash
# axe-core (most popular)
npm install -D @axe-core/cli
axe https://example.com

# Pa11y
npm install -g pa11y
pa11y https://example.com

# Lighthouse
npx lighthouse https://example.com --only-categories=accessibility

# WAVE (browser extension)
# https://wave.webaim.org/extension/
```

### Manual Testing Checklist

- [ ] Keyboard navigation (Tab, Enter, Escape, Arrow keys)
- [ ] Screen reader (NVDA, JAWS, VoiceOver)
- [ ] Color contrast (4.5:1 text, 3:1 UI components)
- [ ] Focus indicators visible
- [ ] Form labels and error messages
- [ ] Alt text for images
- [ ] Semantic HTML
- [ ] ARIA labels where needed

## Complete Audit Guides

**React Accessibility** → `examples/react-a11y.md`
- Component patterns
- Testing with jest-axe
- Common React issues

**Vue Accessibility** → `examples/vue-a11y.md`
- Vue-specific patterns
- Testing strategies

**Testing Guide** → `reference/a11y-testing.md`
- Automated testing
- Screen reader testing
- Keyboard testing

**WCAG Checklist** → `reference/wcag-checklist.md`
- Complete WCAG 2.1 AA criteria
- Common violations
- Fix examples

## Common Issues & Fixes

### 1. Missing Alt Text

**Problem:**
```html
<img src="profile.jpg">
```

**Fix:**
```html
<!-- Decorative image -->
<img src="profile.jpg" alt="">

<!-- Informative image -->
<img src="profile.jpg" alt="Jane Doe, Software Engineer">
```

### 2. Poor Color Contrast

**Problem:**
```css
/* Light gray on white - 2.3:1 ratio (fails) */
color: #999;
background: #fff;
```

**Fix:**
```css
/* Dark gray on white - 4.7:1 ratio (passes AA) */
color: #666;
background: #fff;
```

**Tool:** https://webaim.org/resources/contrastchecker/

### 3. Missing Form Labels

**Problem:**
```html
<input type="email" placeholder="Email">
```

**Fix:**
```html
<label for="email">Email</label>
<input type="email" id="email" name="email">

<!-- Or with aria-label -->
<input type="email" aria-label="Email address">
```

### 4. Button Without Accessible Name

**Problem:**
```html
<button><i class="icon-close"></i></button>
```

**Fix:**
```html
<button aria-label="Close dialog">
  <i class="icon-close" aria-hidden="true"></i>
</button>
```

### 5. No Keyboard Support

**Problem:**
```jsx
<div onClick={handleClick}>Click me</div>
```

**Fix:**
```jsx
<button onClick={handleClick}>Click me</button>

<!-- Or if div required -->
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  Click me
</div>
```

### 6. Missing Focus Indicator

**Problem:**
```css
button:focus {
  outline: none; /* Don't do this! */
}
```

**Fix:**
```css
button:focus-visible {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

### 7. Non-Semantic HTML

**Problem:**
```html
<div class="header">
  <div class="nav">
    <div class="link">Home</div>
  </div>
</div>
```

**Fix:**
```html
<header>
  <nav>
    <a href="/">Home</a>
  </nav>
</header>
```

### 8. Missing Page Title

**Problem:**
```html
<title>Page</title>
```

**Fix:**
```html
<title>User Profile - My App</title>

<!-- React -->
<Helmet>
  <title>User Profile - My App</title>
</Helmet>
```

## ARIA Patterns

### Dialog/Modal

```jsx
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
>
  <h2 id="dialog-title">Confirm Action</h2>
  <p>Are you sure?</p>
  <button onClick={confirm}>Yes</button>
  <button onClick={cancel}>Cancel</button>
</div>
```

### Skip Link

```html
<a href="#main-content" class="skip-link">
  Skip to main content
</a>

<main id="main-content">
  <!-- Content -->
</main>

<style>
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
}
.skip-link:focus {
  top: 0;
}
</style>
```

### Live Region

```jsx
<div aria-live="polite" aria-atomic="true">
  {successMessage}
</div>

<!-- For urgent messages -->
<div aria-live="assertive">
  {errorMessage}
</div>
```

### Tabs

```jsx
<div role="tablist">
  <button
    role="tab"
    aria-selected={activeTab === 'home'}
    aria-controls="home-panel"
  >
    Home
  </button>
</div>
<div role="tabpanel" id="home-panel">
  Content
</div>
```

## Testing

### Automated Testing

```javascript
// Jest + jest-axe
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('has no a11y violations', async () => {
  const { container } = render(<App />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Keyboard Testing

```
Tab - Move forward
Shift+Tab - Move backward
Enter - Activate button/link
Space - Activate button, toggle checkbox
Escape - Close dialog/menu
Arrow keys - Navigate menu/tabs/radio buttons
```

### Screen Reader Testing

**Windows:** NVDA (free)
**Mac:** VoiceOver (built-in, Cmd+F5)
**Mobile:** TalkBack (Android), VoiceOver (iOS)

## Best Practices

✅ **DO:**
- Use semantic HTML (`<button>`, `<nav>`, `<main>`)
- Provide text alternatives for non-text content
- Ensure 4.5:1 color contrast for text
- Support keyboard navigation
- Add focus indicators
- Use ARIA labels when needed
- Test with screen readers
- Test keyboard-only navigation
- Run automated tests (axe, Pa11y)

❌ **DON'T:**
- Use `div` for buttons (`<div onclick>`)
- Remove focus outlines without replacement
- Use color alone to convey information
- Skip heading levels (h1 → h3)
- Use placeholder as label
- Forget alt text on images
- Auto-play audio/video
- Rely only on automated tests

## Tools

**Automated Testing:**
- axe-core, Pa11y, Lighthouse
- jest-axe (React testing)
- eslint-plugin-jsx-a11y

**Manual Testing:**
- WAVE browser extension
- Color contrast checker
- Screen readers (NVDA, VoiceOver)
- Keyboard navigation

**Browser DevTools:**
- Chrome DevTools > Accessibility panel
- Firefox Accessibility Inspector

## Instructions

1. **Run automated audit**
   - axe, Pa11y, or Lighthouse
   - Identify violations

2. **Fix automated issues**
   - Missing alt text
   - Color contrast
   - Form labels
   - ARIA attributes

3. **Manual keyboard test**
   - Tab through page
   - Activate all interactive elements
   - Ensure focus visible

4. **Screen reader test**
   - Navigate with screen reader
   - Verify announcements
   - Check reading order

5. **Verify fixes**
   - Re-run automated tests
   - Test with real users if possible

6. **Prevent regressions**
   - Add jest-axe tests
   - Enable ESLint a11y rules
   - Include in CI/CD

## Constraints

- Must meet WCAG 2.1 Level AA minimum
- Must support keyboard navigation
- Must have visible focus indicators
- Must provide text alternatives for images
- Must maintain 4.5:1 color contrast for text
- Must use semantic HTML where possible
- Must test with screen readers
- Must not rely on color alone
- Should run automated tests in CI
- Must not auto-play audio/video
