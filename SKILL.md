---
name: testing-blocks
description: "Use this when you have made AEM Edge Delivery Services code changes to blocks, scripts, or styles and need to validate them before opening a pull request. Covers unit testing for utilities and logic, browser testing with Playwright, linting, and guidance on what to test and how."
---

# Testing Blocks

This skill guides you through testing code changes in AEM Edge Delivery Services projects. Testing follows a value-versus-cost philosophy: create and maintain tests when the value they bring exceeds the cost of creation and maintenance.

**CRITICAL: Browser validation is MANDATORY. You cannot complete this skill without providing proof of functional testing in a real browser environment.**

## eds-boilerplate project rules (read first)

Rules: `AGENTS.md` (wins over this skill on conflict). Current inventory of
blocks / variants / tokens / section styles / constants: `PROJECT.md`.

- **Tests:** Vitest `blocks/{name}/{name}.test.js` (logic, ARIA, idempotency, empty cells); VRT fixtures `tests/vrt/{name}/fixtures/` × 3 viewports; Storybook stories **reuse** those fixtures (few stories + hand-written `.mdx`); `pnpm run test:a11y` is a release gate.
- **Breakpoints:** design/test at 390 / 767 / 1280; media queries only `768px` / `1024px` (px literal, mobile-first `min-width`). Prefer re-declaring tokens per breakpoint over new `@media` in block CSS.
- **Regressions:** never delete/skip tests or loosen assertions to make them pass; changed VRT baselines must be intentional and called out in the PR. Changes to `styles/`, tokens, `utils/`, `scripts.js`, `constants/` ripple to every block — run the full suite.
- VRT runs in CI only (Linux); local runs are advisory.
- **Before a PR:** `pnpm run verify`, then `node <skills-dir>/code-review/scripts/project-checks.mjs --gates` (diff-aware rule + regression scan (`<skills-dir>` = `.github/skills` for GitHub Copilot, `.claude/skills` for Claude Code)). Branch `{type}/{ticket}-{slug}`.

## Related Skills

- **content-driven-development**: Test content created during CDD serves as the basis for testing
- **building-blocks**: Invokes this skill during Step 5 for comprehensive testing
- **block-collection-and-party**: May provide reference test patterns from similar blocks

## When to Use This Skill

Use this skill:
- ✅ After implementing or modifying blocks
- ✅ After changes to core scripts (scripts.js, delayed.js, aem.js)
- ✅ After style changes (styles.css, lazy-styles.css)
- ✅ After configuration changes that affect functionality
- ✅ Before opening any pull request with code changes

This skill is typically invoked by the **building-blocks** skill during Step 5 (Test Implementation).

## Testing Workflow

Track your progress:

- [ ] Step 1: Run linting and fix issues
- [ ] Step 2: Perform browser validation (MANDATORY)
- [ ] Step 3: Determine if unit tests are needed (optional)
- [ ] Step 4: Run existing tests and verify they pass

## Step 1: Run Linting

**Run linting first to catch code quality issues:**

```bash
pnpm run lint
```

**If linting fails:**
```bash
pnpm run lint:fix
```

**Manually fix remaining issues** that auto-fix couldn't handle.

**Success criteria:**
- ✅ Linting passes with no errors
- ✅ Code follows project standards

**Mark complete when:** `pnpm run lint` passes with no errors

---

## Step 2: Browser Validation (MANDATORY)

**CRITICAL: You must test in a real browser and provide proof.**

### What to Test

Load test content URL(s) in browser and validate:
- ✅ Block/functionality renders correctly
- ✅ Responsive behavior (mobile, tablet, desktop viewports)
- ✅ No console errors
- ✅ Visual appearance matches requirements/acceptance criteria
- ✅ Interactive behavior works (if applicable)
- ✅ All variants render correctly (if applicable)

### How to Test

**Choose the method that makes most sense given your available tools:**

**Option 1: Browser/Playwright MCP (Recommended)**

If you have MCP browser or Playwright tools available, use them directly:
- Navigate to test content URL
- Take accessibility snapshots to inspect rendered content (preferred for interaction)
- Take screenshots at different viewports for visual validation
  - Consider both full-page screenshots and element-specific screenshots of the block being tested
- Interact with elements as needed
- Most efficient for agents with tool access

**Option 2: Playwright automation**

Write one (or more) temporary test scripts to validate functionality with playwright and capture snapshots/screenshots for inspection and validation.

```javascript
// test-my-block.js (temporary - don't commit)
import { chromium } from 'playwright';

async function test() {
  const browser = await chromium.launch({ headless: false });
  const page = await browser.newPage();

  // Navigate and wait for block
  await page.goto('http://localhost:3000/path/to/test');
  await page.waitForSelector('.my-block');

  // Inspect accessibility tree (useful for validating structure)
  const accessibilityTree = await page.accessibility.snapshot();
  console.log('Accessibility tree:', JSON.stringify(accessibilityTree, null, 2));
  
  // Optionally save to file for easier analysis
  await require('fs').promises.writeFile(
    'accessibility-tree.json',
    JSON.stringify(accessibilityTree, null, 2)
  );

  // Test viewports and take screenshots
  await page.setViewportSize({ width: 375, height: 667 });
  await page.screenshot({ path: 'mobile.png', fullPage: true });
  await page.locator('.my-block').screenshot({ path: 'mobile-block.png' });

  await page.setViewportSize({ width: 768, height: 1024 });
  await page.screenshot({ path: 'tablet.png', fullPage: true });
  await page.locator('.my-block').screenshot({ path: 'tablet-block.png' });

  await page.setViewportSize({ width: 1200, height: 800 });
  await page.screenshot({ path: 'desktop.png', fullPage: true });
  await page.locator('.my-block').screenshot({ path: 'desktop-block.png' });

  // Check for console errors
  page.on('console', msg => console.log('Browser:', msg.text()));

  await browser.close();
}

test().catch(console.error);
```

Run: `node test-my-block.js` then delete the script and analyze the resulting artifacts.

**Option 3: Manual browser testing**

Use a standard web browser with dev tools:
1. Navigate to test content: `http://localhost:3000/path/to/test/content`
2. Use browser dev tools responsive mode to test viewports:
   - Mobile: <768px (e.g., 375px)
   - Tablet: 768–1023px (e.g., 768px)
   - Desktop: ≥1024px (e.g., 1280px)
3. Check console for errors at each viewport
4. Take screenshots as proof (browser screenshot tool or dev tools)

### Validation Against Acceptance Criteria

**If acceptance criteria provided (from CDD Step 2):**
- Review each criterion
- Test specific scenarios mentioned
- Verify all criteria are met

**If design/mockup screenshots provided:**
- Compare implementation to design
- Verify visual alignment
- Note any intentional deviations

### Proof of Testing

**You must provide:**
- ✅ Screenshots of test content in browser (at least one viewport)
- ✅ Confirmation no console errors
- ✅ Confirmation acceptance criteria met (if provided)

**Success criteria:**
- ✅ All test content loads and renders correctly
- ✅ Responsive behavior validated across viewports
- ✅ No console errors
- ✅ Screenshots captured as proof
- ✅ Acceptance criteria validated (if provided)

**Mark complete when:** Browser testing complete with screenshots as proof

---

## Step 3: Unit Tests (When Logic Changes)

**Determine if unit tests are needed for this change.**

**Write unit tests when:**
- ✅ New or changed block `decorate` behaviour (DOM, ARIA, interaction)
- ✅ Logic in `utils/` or `scripts/` (transforms, path/locale, API orchestration)
- ✅ Named exports added to blocks (e.g. parsers) that blocks depend on

**Skip unit tests when:**
- ❌ CSS-only changes (use VRT + browser)
- ❌ Visual variants already covered by VRT

**Determine the project's actual layout first, then follow it:** See **`resources/unit-test-architecture.md`** — read the project's `vitest.config` include globs before assuming co-located vs. centralized tests.

| Change type | Test file (co-located example) | Notes |
|-------------|----------------------------------|-------|
| Block | `blocks/{name}/{name}.test.js` | Adjust to the project's actual layout |
| Util | `utils/{name}.test.js` | Adjust to the project's actual layout |
| Script / component | `scripts/__tests__/…` mirroring source | Adjust to the project's actual layout |

**Block example** (co-located, AEM DOM helper, `decorate`):

```javascript
import decorate from './example-block.js';

function createExampleBlock(rows) { /* AEM-shaped rows */ }

it('sets aria-selected on the first item', () => {
  const block = createExampleBlock(['A', 'B']);
  decorate(block);
  expect(block.querySelector('[role="tab"]').getAttribute('aria-selected')).toBe('true');
});
```

**Util example** (input → output; explicit argument rather than a global mock, when supported):

```javascript
import { getLanguagePath } from './example-util.js';

expect(getLanguagePath('/en/page')).toBe('/en');
```

If the project has a shared test-pipeline helper for blocks that need it, use that instead of hand-rolling setup.

```bash
# use whichever test command the project defines
pnpm test   # confirm vitest.config include globs pick up new files
```

**Success criteria:**
- ✅ Test path matches the project's actual Vitest `include` globs
- ✅ Block / util / script pattern matches the project's established layout
- ✅ Pure utils: return-value tests; fetch modules: mock boundary + outcome
- ✅ The project's test command passes

**Mark complete when:** Tests written per `unit-test-architecture.md`, or documented reason to skip

---

## Step 4: Run Existing Tests

**Verify your changes don't break existing functionality:**

```bash
pnpm test
```

**If tests fail:**
1. Read error message carefully
2. Run single test to isolate: `pnpm test -- path/to/test.js`
3. Fix code or update test if expectations changed
4. Re-run full test suite

**CSS convention tests** run as part of `pnpm test` and cover every block
automatically — fix the CSS, don't edit the tests:
- `scripts/__tests__/block-css-conventions.test.js` — `{name}-tokens.css` exists
  and is imported, `{name}.css` uses only `var(--{name}-*)`, every selector
  starts with `.{name}`
- `scripts/__tests__/brand-palette.test.js` — a palette value used in
  `tokens-brand.css` is never referenced directly elsewhere (use the brand token)

**Success criteria:**
- ✅ All existing tests pass
- ✅ No regressions introduced

**Mark complete when:** `pnpm test` passes with no failures

## Project Test Commands: VRT, Storybook, Accessibility

Beyond unit tests and manual browser checks, this project ships three more
gates (see AGENTS.md and `docs/testing/`):

- **Visual regression (VRT)** — `pnpm run test:vrt` (Playwright against the
  shared `tests/vrt/harness.html` fixtures). Regenerate baselines only when a
  visual change is intended: `pnpm run test:vrt:update`. VRT runs in CI, not
  in local hooks.

  **Coverage requirements — VRT must exercise the design, not a stand-in:**
  - **Every variant** gets its own fixture + baseline. If Figma shows N
    variants (default, reverse, compact, …), there are N fixtures — a variant
    with no baseline is an untested variant.
  - **Content and imagery come from Figma.** Fixtures use the same copy and the
    exported/representative images from the Figma frames — not lorem or generic
    placeholders — so the snapshot reflects the real design. Match the frame's
    image aspect ratio and crop.
  - **Interaction and element states** the design defines must be captured too,
    not just the resting layout: `hover`, keyboard `focus`/`:focus-visible`,
    `active`, `disabled`, and any block-specific states (expanded/collapsed,
    selected, error, loading). Drive them in the spec (e.g. `locator.hover()`,
    `.focus()`, force a class/attribute) and snapshot each.
  - Each of the above is verified across the project's viewport projects
    (mobile, tablet, desktop).
- **Storybook** — `pnpm run storybook` previews each block in isolation from
  `blocks/{name}/{name}.stories.js`, reusing the VRT fixtures under
  `tests/vrt/{name}/fixtures/` so sample markup has one source of truth.
- **Accessibility** — `pnpm run test:a11y` runs axe over every story.

For a new or changed block, add or refresh its VRT coverage and story, then run
`pnpm run verify` (lint + unit + VRT) before opening a PR.

## Troubleshooting

For detailed troubleshooting guide, see `resources/troubleshooting.md`.

**Common issues:**

### Tests fail
- Read error message carefully
- Run single test: `pnpm test -- path/to/test.js`
- Fix code or update test

### Linting fails
- Run `pnpm run lint:fix`
- Manually fix remaining issues
- `media-feature-name-value-allowed-list`: media-query widths must be `768px` or
  `1024px` — `var()` is invalid in `@media`, so don't "fix" it with a variable

### Browser tests fail
- Verify dev server running: `aem up --html-folder drafts`
- Check test content exists in `drafts/tmp/`
- Verify URL uses `/tmp/` path: `http://localhost:3000/drafts/tmp/my-block`
- Add waits: `await page.waitForSelector('.block')`

## Resources

- **Unit Testing:** `resources/unit-testing.md` - Complete guide to writing and maintaining unit tests
- **Unit Test Architecture:** `resources/unit-test-architecture.md` - Two-layer model (pure logic + narrow integration); anti-patterns
- **Troubleshooting:** `resources/troubleshooting.md` - Solutions to common testing issues
- **Vitest Setup:** `resources/vitest-setup.md` - One-time configuration guide
- **Testing Philosophy:** `resources/testing-philosophy.md` - Guide on what and how to test

## Integration with Building Blocks Skill

The **building-blocks** skill invokes this skill during Step 5 (Test Implementation).

**Inputs received from building-blocks:**
- Block name being tested
- Test content URL(s) (from CDD Step 4)
- Any variants that need testing
- Screenshots of existing implementation/design/mockup to verify against (if provided)
- Acceptance criteria to verify (from CDD Step 2)

**Expected outputs to return to building-blocks:**
- ✅ Confirmation all testing steps complete
- ✅ Screenshots from browser testing as proof
- ✅ Confirmation linting passes
- ✅ Confirmation tests pass
- ✅ Any issues discovered and resolved
