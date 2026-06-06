---
name: web-accessibility
description: Make HTML websites accessible and WCAG-compliant. Use this skill whenever the user mentions accessibility, a11y, WCAG, screen readers, keyboard navigation, color contrast, or ARIA in the context of HTML pages. Also trigger when the user asks to audit, fix, or improve the accessibility of any HTML or CSS code, or mentions making a site usable for people with disabilities, blind users, motor impairments, or low vision — even if they don't use the word "accessibility". If the user shares any HTML and wants it improved or audited in any way, check if accessibility improvements are warranted and apply this skill proactively.
---

# HTML Web Accessibility Skill

This skill guides systematic auditing and remediation of HTML pages and snippets to meet **WCAG 2.2 AA** (the current legal and industry standard).

> **Project context**: This is an **Astro** project (Al-Sultan & Khalaf Trading Co). All guidance below includes Astro-specific considerations alongside generic HTML/CSS/JS patterns.

---

## Step 1 — Understand Scope

Before touching any code, clarify:

1. **What's the target?** A whole page, a section, or a snippet?
2. **What's the standard?** Default to WCAG 2.2 AA unless the user specifies AAA or Section 508.
3. **Is there existing automated tooling?** (axe DevTools, Lighthouse) — note it if present.

If the user just pastes HTML and asks to "make it accessible", proceed with a full audit and fix pass without asking — annotate every change with an HTML comment explaining the fix.

---

## Step 1.5 — Astro-Specific Considerations

This project uses **Astro** (`.astro` files), which affects how accessibility patterns are implemented:

### Scoped vs Global Styles

- Astro `<style>` tags are **scoped by default** — classes only apply to the component that defines them.
- Accessibility utility classes (`.sr-only`, `.skip-link`, `:focus-visible`) **must be global** so they work across all components.
- Use `<style is:global>` or place global accessibility styles in `src/styles/global.css`.

```astro
<!-- ❌ Bad: scoped style — .sr-only won't work in child components -->
<style>
  .sr-only { ... }
</style>

<!-- ✅ Good: global style available everywhere -->
<style is:global>
  .sr-only { ... }
</style>

<!-- ✅ Best: define in src/styles/global.css (imported by Layout.astro) -->
```

### Component Boundaries

- **Skip-link** must be placed in `src/layouts/Layout.astro` (before the `<slot />`), not inside `header.astro` or any other component.
- **Landmark regions** (`<header>`, `<nav>`, `<main>`, `<footer>`) should be verified at the **page assembly level** (`src/pages/index.astro`), not just within individual components.
- The `<main>` element wrapping content needs `id="main-content"` and `tabindex="-1"` for skip-link targeting — verify this in `src/pages/index.astro`.

### Client-Side Scripts

- Astro component `<script>` tags run on the client by default (no special directive needed for non-framework components).
- Scripts in `.astro` components are bundled and executed once — use `document.getElementById()` or `document.querySelector()` as normal.
- For accessibility JS (focus trapping, keyboard handlers), place the script in the component that owns the interactive element (e.g., mobile menu script in `header.astro`).

---

## Step 1.6 — Astro Components to Audit

When auditing this project, check these specific files for each accessibility area:

| What to check                                  | File to inspect                                                      |
| ---------------------------------------------- | -------------------------------------------------------------------- |
| `lang` attribute on `<html>`                   | `src/layouts/Layout.astro`                                           |
| `<title>` and meta description                 | `src/layouts/Layout.astro`                                           |
| Skip-link (before `<slot />`)                  | `src/layouts/Layout.astro`                                           |
| `.sr-only` and `.skip-link` classes            | `src/styles/global.css`                                              |
| `:focus-visible` styles                        | `src/styles/global.css`                                              |
| `prefers-reduced-motion` global rule           | `src/styles/global.css`                                              |
| Nav landmarks, `aria-label`                    | `src/components/header.astro`                                        |
| `aria-current="page"` on active link           | `src/components/header.astro`                                        |
| Mobile menu focus trap + Escape key            | `src/components/header.astro` `<script>`                             |
| External link sr-only text + SVG `aria-hidden` | `src/components/header.astro` (Shop link)                            |
| Heading hierarchy (`<h1>` usage)               | `src/components/hero.astro`                                          |
| Image `alt` text                               | `src/components/hero.astro`, `src/components/section/About.astro`    |
| CSS animation reduced-motion handling          | `src/components/hero.astro` (keyframes)                              |
| Section landmarks and `aria-label`             | `src/components/section/About.astro`                                 |
| Counter animation reduced-motion               | `src/components/section/About.astro` `<script>` (✅ already handled) |

> **Rule**: When the user asks to audit or fix accessibility, always walk through this table file-by-file rather than relying on generic checks alone.

---

## Step 2 — Audit Checklist

Work through the POUR principles and flag every issue with its WCAG criterion number.

### 2.1 Perceivable

| Check                                          | WCAG   | What to look for                                                |
| ---------------------------------------------- | ------ | --------------------------------------------------------------- |
| Images have alt text                           | 1.1.1  | `<img>` missing `alt`; decorative images need `alt=""`          |
| Videos have captions                           | 1.2.2  | `<video>` without `<track kind="captions">`                     |
| Color is not the only differentiator           | 1.4.1  | Errors shown only in red; links only styled by color            |
| Text contrast ≥ 4.5:1 (normal) / 3:1 (large)   | 1.4.3  | Foreground vs background; includes placeholder text             |
| UI component contrast ≥ 3:1                    | 1.4.11 | Input borders, icon-only buttons                                |
| Text resizes to 200% without horizontal scroll | 1.4.4  | Fixed `px` widths on text containers                            |
| No content flashes > 3/sec                     | 1.4.2  | Animations, GIFs, auto-playing video                            |
| Reflow at 400% zoom (320px viewport)           | 1.4.10 | Content hidden or overflowing horizontally                      |
| Spacing overrides don't break layout           | 1.4.12 | Hardcoded `line-height` or `letter-spacing`                     |
| Animations respect `prefers-reduced-motion`    | 2.3.3  | CSS keyframes and JS animations without reduced-motion handling |

### 2.2 Operable

| Check                                       | WCAG           | What to look for                                                      |
| ------------------------------------------- | -------------- | --------------------------------------------------------------------- |
| All interactive elements keyboard-reachable | 2.1.1          | `<div onclick>` without `tabindex`/`role`; missing focusable elements |
| No keyboard traps                           | 2.1.2          | Modals and **mobile slide-out menus** that swallow Tab key            |
| Focus visible                               | 2.4.7 / 2.4.11 | `outline: none` with no replacement; low-contrast focus rings         |
| Skip navigation link present                | 2.4.1          | No "Skip to main content" at top of page                              |
| Page has a meaningful `<title>`             | 2.4.2          | Generic, empty, or duplicated `<title>`                               |
| Link purpose is clear from text             | 2.4.4          | "Click here", "Read more" with no surrounding context                 |
| Heading hierarchy is logical                | 2.4.6          | Skipped levels (h1 → h3); headings used purely for styling            |
| Touch targets ≥ 24×24 CSS px                | 2.5.8          | Small icon buttons, inline close buttons                              |
| No seizure-risk animation                   | 2.3.1          | 3+ flashes per second                                                 |
| Session timeouts have warnings              | 2.2.1          | Expiry with no advance notice or extension option                     |
| Active page indicated to assistive tech     | 4.1.2          | Visual-only active states without `aria-current="page"`               |

### 2.3 Understandable

| Check                                           | WCAG          | What to look for                                                |
| ----------------------------------------------- | ------------- | --------------------------------------------------------------- |
| `lang` attribute on `<html>`                    | 3.1.1         | Missing or incorrect language code                              |
| Every input has a label                         | 1.3.1 / 3.3.2 | Placeholder-only labels; `<input>` with no associated `<label>` |
| Error messages are descriptive and associated   | 3.3.1 / 3.3.3 | "Invalid input" only; errors not linked via `aria-describedby`  |
| Instructions don't rely on shape/position/color | 1.3.3         | "Click the green button on the right"                           |
| Navigation is consistent across pages           | 3.2.3         | Nav order or structure changes between pages                    |
| No unexpected context change on focus           | 3.2.1         | Form submits on focus; new tabs open without warning            |

### 2.4 Robust

| Check                                        | WCAG  | What to look for                                           |
| -------------------------------------------- | ----- | ---------------------------------------------------------- |
| Semantic HTML used correctly                 | 4.1.1 | `<div>` as `<button>`; duplicate `id` values               |
| ARIA only added when native HTML can't do it | 4.1.2 | ARIA overriding correct native semantics; stale labels     |
| Dynamic content uses live regions            | 4.1.3 | Toast notifications, loaders, search results not announced |
| Status messages are programmatically exposed | 4.1.3 | Form success messages not in an `aria-live` region         |

---

## Step 3 — Fix Patterns

### Page Shell — Always Start Here

> **⚠️ Project check**: Verify that `src/layouts/Layout.astro` includes **all** of the following. If any are missing, add them.

```html
<!DOCTYPE html>
<!-- ✅ lang attribute is required (WCAG 3.1.1) -->
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <!-- ✅ Descriptive, unique title per page (WCAG 2.4.2) -->
    <title>Contact Us — Acme Co</title>
  </head>
  <body>
    <!-- ✅ Skip link — very first focusable element (WCAG 2.4.1) -->
    <a href="#main-content" class="skip-link">Skip to main content</a>

    <!-- ✅ Landmark regions help screen reader users navigate (WCAG 1.3.1) -->
    <header role="banner">...</header>
    <nav aria-label="Main navigation">...</nav>

    <!-- ✅ tabindex="-1" lets JS focus the element after skip-link activation -->
    <main id="main-content" tabindex="-1">...</main>

    <footer role="contentinfo">...</footer>
  </body>
</html>
```

**In this Astro project**, the Layout renders a `<slot />` inside `<body>`. The skip-link must be placed **before the `<slot />`** in `Layout.astro`, and the `<main>` element in `index.astro` must have `id="main-content"` and `tabindex="-1"`:

```astro
<!-- src/layouts/Layout.astro -->
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <slot />
</body>

<!-- src/pages/index.astro -->
<main id="main-content" tabindex="-1">
  <Hero />
  <About />
</main>
```

### Semantic HTML — Always Prefer Native Elements

```html
<!-- ❌ Bad: div has no role, no keyboard access, no focus -->
<div class="btn" onclick="submit()">Submit</div>

<!-- ✅ Good: button is keyboard-accessible, focusable, and announced correctly -->
<button type="submit">Submit</button>

<!-- ❌ Bad: span used as a heading -->
<span class="big-text">Section Title</span>

<!-- ✅ Good: heading communicates structure to screen readers -->
<h2>Section Title</h2>

<!-- ❌ Bad: div used as a list -->
<div class="list">
  <div>Item one</div>
  <div>Item two</div>
</div>

<!-- ✅ Good: semantic list -->
<ul>
  <li>Item one</li>
  <li>Item two</li>
</ul>
```

### Images

```html
<!-- ✅ Informative image: describe what it conveys, not what it looks like -->
<img
  src="q3-chart.png"
  alt="Bar chart: revenue grew 40% in Q3 2024 vs Q3 2023"
/>

<!-- ✅ Decorative image: empty alt tells screen reader to skip it -->
<img src="divider.svg" alt="" />

<!-- ✅ Icon-only button: label the button, hide the icon -->
<button aria-label="Close dialog">
  <svg aria-hidden="true" focusable="false" width="16" height="16">...</svg>
</button>

<!-- ✅ Image that IS the link: alt describes the destination -->
<a href="/home">
  <img src="logo.png" alt="Acme Co — Home" />
</a>
```

### Forms and Labels

```html
<!-- ✅ Explicit label — preferred method -->
<label for="email">Email address</label>
<input id="email" type="email" autocomplete="email" required />

<!-- ✅ Grouped radio/checkbox inputs need a fieldset + legend -->
<fieldset>
  <legend>Preferred contact method</legend>
  <label><input type="radio" name="contact" value="email" /> Email</label>
  <label><input type="radio" name="contact" value="phone" /> Phone</label>
</fieldset>

<!-- ✅ Search input with no visible label: use aria-label -->
<input type="search" aria-label="Search products" placeholder="Search…" />

<!-- ❌ Bad: placeholder is NOT a label — disappears on input, low contrast -->
<input type="text" placeholder="Enter your name" />

<!-- ✅ Good: visible label + optional placeholder for hint text -->
<label for="name">Full name</label>
<input
  id="name"
  type="text"
  placeholder="e.g. Jane Smith"
  autocomplete="name"
/>

<!-- ✅ Error state: associate the error message with the input -->
<label for="email2">Email address</label>
<input
  id="email2"
  type="email"
  aria-describedby="email-error"
  aria-invalid="true"
/>
<!-- role="alert" announces the error immediately when injected -->
<p id="email-error" role="alert">
  Enter a valid email address, e.g. you@example.com
</p>
```

### Links

```html
<!-- ❌ Bad: no context -->
<a href="/report.pdf">Click here</a>
<a href="/blog/post">Read more</a>

<!-- ✅ Good: purpose clear from link text alone -->
<a href="/report.pdf">Download Q3 2024 Annual Report (PDF, 2 MB)</a>
<a href="/blog/post">Read more about our accessibility work</a>

<!-- ✅ If you must use short text, add sr-only context -->
<a href="/blog/post">
  Read more
  <span class="sr-only"> about our accessibility work</span>
</a>

<!-- ✅ External link: warn users it opens a new tab -->
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Example site
  <span class="sr-only">(opens in new tab)</span>
</a>
```

### Links with SVG Icons

> **⚠️ Project check**: The **Shop** link in `src/components/header.astro` opens in a new tab and uses an SVG icon as a visual indicator. This pattern requires two fixes:

```html
<!-- ❌ Bad: SVG has no aria-hidden, no sr-only text for screen readers -->
<a href="/shop" target="_blank" rel="noopener noreferrer">
  Shop
  <svg width="14" height="14" viewBox="0 0 24 24">...</svg>
</a>

<!-- ✅ Good: SVG hidden from screen readers, sr-only announces new tab -->
<a href="/shop" target="_blank" rel="noopener noreferrer">
  Shop
  <svg
    aria-hidden="true"
    focusable="false"
    width="14"
    height="14"
    viewBox="0 0 24 24"
  >
    ...
  </svg>
  <span class="sr-only">(opens in new tab)</span>
</a>
```

**Always check**: Any `<svg>` inside an interactive element (`<a>`, `<button>`) that serves as a decorative icon must have `aria-hidden="true"` and `focusable="false"`. If the SVG conveys meaning (e.g., it IS the link's content), give it a `role="img"` and `aria-label` instead.

### Active Navigation — `aria-current`

> **⚠️ Project check**: The active nav link in `src/components/header.astro` uses a CSS modifier class (`header__nav-link--active`) for visual styling, but has **no programmatic indication** for screen readers.

```html
<!-- ❌ Bad: visual-only active state — screen readers don't know this is the current page -->
<a href="/" class="header__nav-link header__nav-link--active">Home</a>

<!-- ✅ Good: aria-current="page" tells screen readers "you are here" -->
<a
  href="/"
  class="header__nav-link header__nav-link--active"
  aria-current="page"
  >Home</a
>
```

In Astro, you can compute the active page dynamically:

```astro
---
const currentPath = Astro.url.pathname;
---
<a
  href="/"
  class:list={["header__nav-link", { "header__nav-link--active": currentPath === "/" }]}
  aria-current={currentPath === "/" ? "page" : undefined}
>
  Home
</a>
```

### Focus Styles (CSS)

> **⚠️ Project check**: `src/styles/global.css` currently has **no `:focus-visible` rule**. The button reset (`border: none; background: none;`) means browser default focus indicators may not render properly. You **must** add `:focus-visible` styles to `global.css`.

```css
/* ❌ Never suppress focus without a replacement */
* {
  outline: none;
} /* BAD */

/* ✅ Replace the default outline with a visible, high-contrast ring */
/* Add this to src/styles/global.css */
:focus-visible {
  outline: 2px solid #005fcc;
  outline-offset: 2px;
  border-radius: 2px;
}

/* ✅ For elements on dark backgrounds (hero, about-us sections) */
.hero :focus-visible,
.about-us :focus-visible {
  outline-color: #ffffff;
}
```

### Skip Navigation (CSS)

> **⚠️ Project check**: These styles must be in `src/styles/global.css` (not a scoped component style) so the skip-link in `Layout.astro` is styled correctly.

```css
/* Visually hidden until focused — appears for keyboard users */
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000000;
  color: #ffffff;
  padding: 8px 16px;
  text-decoration: none;
  z-index: 9999;
  font-weight: bold;
}
.skip-link:focus {
  top: 0;
}
```

### Visually Hidden Utility Class

> **⚠️ Project check**: This must be in `src/styles/global.css` so it works across all components (header, hero, sections, etc.).

```css
/* Hides content visually but keeps it available to screen readers */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### Reduced Motion (CSS)

> **⚠️ Project check**: `src/styles/global.css` has **no `prefers-reduced-motion` rule**. The hero section in `src/components/hero.astro` uses CSS keyframe animations (`heroFadeUp`) that always run, ignoring user motion preferences. The counter animation in `src/components/section/About.astro` correctly handles this in JS — cite it as the good example.

Add this global rule to `src/styles/global.css`:

```css
/* ✅ Respect user's motion preferences globally (WCAG 2.3.3) */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

This is a **blanket safety net**. For finer control per-component, you can also add targeted media queries:

```css
/* In hero.astro scoped styles */
@media (prefers-reduced-motion: reduce) {
  .hero__subtitle,
  .hero__title,
  .hero__actions {
    animation: none;
  }
}
```

### Live Regions for Dynamic Content

```html
<!-- ✅ Polite: announces after current speech finishes (toasts, confirmations) -->
<div
  id="status-msg"
  aria-live="polite"
  aria-atomic="true"
  class="sr-only"
></div>

<!-- ✅ Assertive: interrupts immediately (critical errors only) -->
<div id="error-msg" role="alert" class="sr-only"></div>
```

```js
// Inject text to trigger announcement
document.getElementById("status-msg").textContent =
  "Your changes have been saved.";
```

### Modal Dialogs

```html
<div
  id="dialog"
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  hidden
>
  <h2 id="dialog-title">Confirm deletion</h2>
  <p>Are you sure you want to delete this item? This cannot be undone.</p>
  <button id="dialog-confirm">Delete</button>
  <button id="dialog-cancel">Cancel</button>
</div>
```

```js
const focusableSelectors =
  'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])';

function openDialog(dialog, trigger) {
  dialog.removeAttribute("hidden");
  // Move focus to first focusable element inside dialog
  dialog.querySelector(focusableSelectors)?.focus();

  // Trap Tab and Shift+Tab within the dialog
  dialog.addEventListener("keydown", trapFocus);
  // Store the trigger so we can restore focus on close
  dialog._trigger = trigger;
}

function closeDialog(dialog) {
  dialog.setAttribute("hidden", "");
  dialog.removeEventListener("keydown", trapFocus);
  // Return focus to the element that opened the dialog
  dialog._trigger?.focus();
}

function trapFocus(e) {
  if (e.key !== "Tab") return;
  const focusable = [...this.querySelectorAll(focusableSelectors)];
  const first = focusable[0];
  const last = focusable[focusable.length - 1];
  if (e.shiftKey && document.activeElement === first) {
    e.preventDefault();
    last.focus();
  } else if (!e.shiftKey && document.activeElement === last) {
    e.preventDefault();
    first.focus();
  }
}
```

### Mobile Slide-Out Navigation Menu

> **⚠️ Project check**: The mobile menu in `src/components/header.astro` currently toggles open/close and updates `aria-expanded`, but is **missing focus trapping, Escape key handling, and focus management**. A slide-out nav that overlays the page must be treated like a dialog for keyboard accessibility.

The current script only does this:

```js
// ❌ Incomplete: no focus trap, no Escape, no focus management
menuToggle.addEventListener("click", () => {
  const isOpen = mainNav.classList.toggle("header__nav--open");
  menuToggle.classList.toggle("header__menu-toggle--active");
  menuToggle.setAttribute("aria-expanded", String(isOpen));
});
```

Replace with a fully accessible version:

```js
const menuToggle = document.getElementById("menu-toggle");
const mainNav = document.getElementById("main-nav");

if (menuToggle && mainNav) {
  const focusableSelectors =
    'a[href], button, input, select, textarea, [tabindex]:not([tabindex="-1"])';

  function openMenu() {
    mainNav.classList.add("header__nav--open");
    menuToggle.classList.add("header__menu-toggle--active");
    menuToggle.setAttribute("aria-expanded", "true");

    // Move focus to the first nav link
    const firstLink = mainNav.querySelector(focusableSelectors);
    firstLink?.focus();

    // Listen for Escape and Tab
    document.addEventListener("keydown", handleMenuKeydown);
  }

  function closeMenu() {
    mainNav.classList.remove("header__nav--open");
    menuToggle.classList.remove("header__menu-toggle--active");
    menuToggle.setAttribute("aria-expanded", "false");

    // Return focus to the toggle button
    menuToggle.focus();

    document.removeEventListener("keydown", handleMenuKeydown);
  }

  function handleMenuKeydown(e) {
    if (e.key === "Escape") {
      closeMenu();
      return;
    }

    if (e.key === "Tab") {
      const focusable = [...mainNav.querySelectorAll(focusableSelectors)];
      // Include the toggle button in the focus trap
      focusable.unshift(menuToggle);
      const first = focusable[0];
      const last = focusable[focusable.length - 1];

      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  }

  menuToggle.addEventListener("click", () => {
    const isOpen = mainNav.classList.contains("header__nav--open");
    if (isOpen) {
      closeMenu();
    } else {
      openMenu();
    }
  });
}
```

**Key differences from modal dialogs:**

- The toggle button is **included** in the focus trap (it stays visible).
- Escape closes the menu and returns focus to the toggle.
- Focus moves to the first nav link on open (not the first focusable element generally).

### Tables

```html
<!-- ✅ Data table with proper headers -->
<table>
  <!-- caption is the accessible name for the table -->
  <caption>
    Q3 2024 Sales by Region
  </caption>
  <thead>
    <tr>
      <!-- scope tells screen readers which cells this header applies to -->
      <th scope="col">Region</th>
      <th scope="col">Revenue</th>
      <th scope="col">Growth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">North</th>
      <!-- row header -->
      <td>£1.2 M</td>
      <td>+12%</td>
    </tr>
  </tbody>
</table>

<!-- ❌ Never use a table for layout — use CSS Grid or Flexbox instead -->
```

### Color Contrast

Normal text (< 18pt / < 14pt bold): **4.5:1 minimum**
Large text (≥ 18pt / ≥ 14pt bold): **3:1 minimum**
UI components and icons: **3:1 minimum**
Placeholder text: **4.5:1 minimum** (counts as text)

Check with: browser DevTools colour picker, or [contrast-ratio.com](https://contrast-ratio.com).

When fixing, prefer darkening text or lightening the background. Always re-check both light and dark mode variants.

---

## Step 4 — Output Format

When the user provides HTML to audit:

1. **Summary table** — each issue, WCAG criterion, severity, and one-line fix description.
2. **Remediated HTML** — corrected code with `<!-- accessibility: ... -->` comments on every changed line.
3. **Remaining manual checks** — list what tools can't verify (cognitive load, real screen reader flow, device testing).

**Severity levels:**

- **Critical** — Blocks a feature entirely (e.g., form submit button unreachable by keyboard)
- **Serious** — Causes significant difficulty (e.g., unlabelled input)
- **Moderate** — Causes some friction (e.g., low contrast on secondary text)
- **Minor** — Best-practice improvement (e.g., redundant ARIA attribute)

---

## Step 5 — Testing Recommendations

Always tell the user to verify with:

| Tool                                                         | What it catches                   |
| ------------------------------------------------------------ | --------------------------------- |
| [axe DevTools](https://www.deque.com/axe/) browser extension | ~57% of WCAG issues automatically |
| Chrome Lighthouse (DevTools → Lighthouse tab)                | Quick accessibility score         |
| Keyboard-only (Tab, Shift+Tab, Enter, Space, Arrow keys)     | Focus order, traps, visibility    |
| NVDA + Firefox or JAWS + Chrome (Windows)                    | Screen reader announcements       |
| VoiceOver + Safari (macOS / iOS)                             | Apple ecosystem behaviour         |
| TalkBack (Android Chrome)                                    | Mobile screen reader              |
| Browser zoom to 200% and 400%                                | Text resize and reflow            |
| Windows High Contrast Mode                                   | Colour-independent rendering      |

Automated tools catch at most ~57% of issues. Keyboard and screen reader testing is essential.

---

## Common Anti-Patterns — Always Fix These

- `outline: none` with no replacement → add `:focus-visible` style
- No `:focus-visible` in `global.css` when buttons are reset → add global focus ring
- `<div onclick>` or `<span onclick>` → replace with `<button>` or `<a>`
- `<input>` with only a `placeholder` and no `<label>` → add an explicit `<label for="...">`
- Error messages shown only in red with no text → add `role="alert"` and `aria-describedby`
- `<img>` missing `alt` → add descriptive alt or `alt=""` for decorative
- `tabindex` values greater than 0 → use only `0` or `-1`
- `aria-label` that duplicates adjacent visible text → remove it; it's redundant
- Links opening new tabs without warning → add `<span class="sr-only">(opens in new tab)</span>`
- SVG icons inside links/buttons with no `aria-hidden="true"` → add `aria-hidden="true" focusable="false"`
- Layout tables → replace with CSS Grid or Flexbox
- `<b>` or `<i>` used for semantics → use `<strong>` or `<em>` respectively
- Empty `<button>` or `<a>` with only an icon → add `aria-label`
- Visual-only active navigation state → add `aria-current="page"`
- CSS animations without `prefers-reduced-motion` handling → add global or per-component media query
- Mobile menu with no focus trap → implement Escape key + Tab trapping + focus management
- Accessibility utility classes (`.sr-only`, `.skip-link`) in scoped Astro styles → move to `global.css`

---

## WCAG 2.2 Quick Reference

| Criterion | Level | Topic                                  |
| --------- | ----- | -------------------------------------- |
| 1.1.1     | A     | Non-text content (alt text)            |
| 1.3.1     | A     | Info and relationships (semantic HTML) |
| 1.3.3     | A     | Sensory characteristics                |
| 1.4.1     | A     | Use of colour                          |
| 1.4.3     | AA    | Text contrast                          |
| 1.4.4     | AA    | Resize text                            |
| 1.4.10    | AA    | Reflow                                 |
| 1.4.11    | AA    | Non-text contrast                      |
| 1.4.12    | AA    | Text spacing                           |
| 2.1.1     | A     | Keyboard accessible                    |
| 2.1.2     | A     | No keyboard trap                       |
| 2.3.1     | A     | Three flashes or below threshold       |
| 2.3.3     | AAA   | Animation from interactions            |
| 2.4.1     | A     | Bypass blocks (skip nav)               |
| 2.4.2     | A     | Page titled                            |
| 2.4.3     | A     | Focus order                            |
| 2.4.4     | A     | Link purpose                           |
| 2.4.7     | AA    | Focus visible                          |
| 2.4.11    | AA    | Focus appearance (WCAG 2.2)            |
| 2.5.8     | AA    | Target size minimum (WCAG 2.2)         |
| 3.1.1     | A     | Language of page                       |
| 3.2.1     | A     | On focus (no unexpected change)        |
| 3.3.1     | A     | Error identification                   |
| 3.3.2     | A     | Labels or instructions                 |
| 4.1.1     | A     | Valid HTML, no duplicate IDs           |
| 4.1.2     | A     | Name, role, value                      |
| 4.1.3     | AA    | Status messages                        |
