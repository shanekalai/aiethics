---
name: siahus-design
description: Frontend design skill for the siahus WordPress theme — UI component registry, visual regression testing with Playwright, deployment verification, and WooCommerce integration. Use this skill for ANY frontend design, styling, UI coding, shop page, product page, template, CSS, or visual layout task on the siahus project. Also use when debugging visual regressions, filter behavior, category display issues, or WooCommerce template problems. MANDATORY before any UI work.
---

# siahus Design Skill

Comprehensive frontend design skill covering the siahus WordPress theme aesthetic, component library, Playwright visual testing pipeline, deployment workflow, and WooCommerce integration patterns.

## When to Read Reference Files

| Task | Reference to Read |
|------|-------------------|
| Setting up or running Playwright tests | [playwright-visual-testing.md](references/playwright-visual-testing.md) |
| WooCommerce template, product, or category issues | [woocommerce-gotchas.md](references/woocommerce-gotchas.md) |
| All other frontend/UI/CSS tasks | Continue reading this file |

---

## Design Thinking

Before coding any UI, commit to a clear aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: **Organic/natural + luxury/refined** — sage greens, warm creams, Cormorant Garamond display font, gentle animations, foliage overlays.
- **Constraints**: Vanilla CSS + JS (no frameworks), WordPress/PHP templates, mobile-first responsive.
- **Differentiation**: What makes this memorable? Intentional design > generic patterns.

**Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.**

### Aesthetic Guidelines

- **Typography**: `--font-display` (Cormorant Garamond) for headings, `--font-sans` (Inter) for body. Never introduce new fonts without discussion.
- **Color**: CSS custom properties exclusively. `--color-sage` for primary, `--color-coral` for accents, `--color-cream` for backgrounds. Never hardcode hex values.
- **Motion**: Coordinated page-load reveals with stagger, scroll-triggered animations, hover lifts on cards. Use `--transition-fast/medium/slow` variables.
- **Spatial Composition**: Generous negative space. Cards have subtle shadows and border radius. Grid layouts with responsive breakpoints.

**NEVER use generic AI aesthetics**: overused fonts (Arial, Roboto), cliched purple gradients, cookie-cutter layouts. Every design should feel like the siahus brand.

---

## Source of Truth

### File Authority

**Canonical source for all deployed theme files:**
```
staging/extracted-theme/wp-content/themes/siahus/
```
**NEVER deploy from other folders.** Stale copies caused a production incident (2026-03-09).

### Single Source of Truth Principle

> **ALWAYS use common source files for ALL global website assets.**

- Global page heroes → `.page-hero` in `pages.css`
- Filter bars → `.lc-filters`, `.filter-row` in `learning-center.css`
- Custom dropdowns → `.filter-btn`, `.filter-dropdown` in `learning-center.css`
- Product cards → `.product-card` in `components.css`
- Buttons → `.btn` variants in `components.css`
- All colors/spacing → CSS custom properties in `style.css`

**If a component exists, REUSE it. Never recreate or duplicate styles.**

### Category Display Order

The header nav, shop filter dropdown, and footer shop links all share an explicit category order. When adding a new category, update three locations:

1. **WordPress nav menu** (wp-admin → Appearance → Menus)
2. **`template-shop.php`** — `$cat_display_order` array
3. **`footer.php`** — `$footer_cat_order` array

See [woocommerce-gotchas.md § Category Display Order](references/woocommerce-gotchas.md) for the full current order and details.

---

## Pre-Deploy Checklist

Before deploying ANY file to the server:

1. **Verify you are editing the `extracted-theme/` copy** — not a stale copy elsewhere
2. **Read the current server version first** — SSH cat the file to confirm your local copy matches
3. **If they diverge**: the server version is authoritative. Pull it down, make changes on top
4. **Deploy via SCP** to the theme path on server
5. **Clear ALL caches** (both are required):
   ```bash
   # Valkey object cache (holds WordPress term cache, options, transients)
   docker exec valkey valkey-cli FLUSHALL
   # Nginx FastCGI page cache
   docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'
   ```
6. **Visually verify** every affected page (see Post-Deploy Verification below)

### Post-Deploy Verification — Hybrid Approach

> **Assertions first, screenshots last.** Assertions find bugs cheaply (~50 tokens each). Screenshots confirm aesthetics expensively (~2000+ tokens each). Use assertions to iterate, screenshots to confirm.

After every deploy, follow this two-phase verification:

#### Phase 1: Structural Assertions (fast, cheap — finds 80% of bugs)

Run Playwright assertions to verify layout, z-index stacking, visibility, dimensions, and CSS values. These return text results, not images, so they cost minimal tokens.

```bash
# Run structural assertions only (skips screenshots)
npx playwright test tests/interactive/ --grep "assert" --project=mobile --reporter=list
```

**What assertions verify:**
- [ ] Element visibility (`toBeVisible()` — checks actual rendering, not just CSS)
- [ ] Z-index stacking (nav panel above overlay, dropdowns above content)
- [ ] Element dimensions and position (`boundingBox()` — not off-screen, correct size)
- [ ] CSS values (background-color, position, font-family, overflow)
- [ ] Body scroll lock (overflow: hidden when modals/nav open)
- [ ] Element counts (menu links present, filter items rendered)
- [ ] Close/dismiss behavior (Escape, overlay click, close button)
- [ ] Responsive breakpoints (hamburger hidden at desktop, visible at mobile)

**Example assertion patterns:**
```typescript
// Z-index stacking check (catches the nav panel bug)
const navZ = await getZIndex(page.locator('.mobile-nav-panel'));
const overlayZ = await getZIndex(page.locator('.mobile-overlay'));
expect(navZ).toBeGreaterThan(overlayZ);

// Visual stacking check (is element actually on top and clickable?)
const isClickable = await element.evaluate(el => {
  const rect = el.getBoundingClientRect();
  const topEl = document.elementFromPoint(rect.x + rect.width / 2, rect.y + rect.height / 2);
  return el.contains(topEl);
});
expect(isClickable).toBe(true);

// CSS value check
await expect(navPanel).toHaveCSS('position', 'fixed');
await expect(navPanel).toHaveCSS('background-color', /rgb/); // not transparent
```

If assertions fail, fix the code and re-run assertions. Do NOT take screenshots during debugging — it wastes tokens.

#### Phase 2: Visual Confirmation Screenshots (run AFTER all assertions pass)

Once structural assertions pass, take screenshots for final visual sign-off.

```bash
# Run screenshots only (visual confirmation after assertions pass)
npx playwright test tests/interactive/ --grep "screenshot" --project=mobile --reporter=list
```

**For static pages (no interaction needed):**
```bash
msedge --headless --screenshot=screenshots/{page}-desktop.png --window-size=1400,900 https://siahus.cloudbranch.co/{page}/
msedge --headless --screenshot=screenshots/{page}-mobile.png --window-size=375,812 https://siahus.cloudbranch.co/{page}/
```
For JS-dependent pages add `--virtual-time-budget=5000`.

**For interactive UI (needs clicks before capture):**
Edit the `SCENARIOS` array in `tests/interactive/take-screenshot.ts`, then run:
```bash
npx playwright test tests/interactive/take-screenshot.ts --project=mobile
```

**If Node.js is not installed**, see [playwright-visual-testing.md § Prerequisites](references/playwright-visual-testing.md) for setup instructions. Test files are pre-built — only install dependencies.

#### What to verify after every deploy
- [ ] Page hero renders correctly (`.page-hero` from `pages.css`)
- [ ] Filter bars function (dropdowns open, search works, results filter)
- [ ] Category order matches across header, dropdown, and footer
- [ ] Product/article cards display properly (images, text, hover effects)
- [ ] Foliage overlays visible and z-indexed correctly
- [ ] Mobile responsive — check at 640px and 1024px breakpoints
- [ ] No duplicate `<main>` nesting (header.php already opens `<main>`)

### Visual Proof of Work — MANDATORY

**UI work is NOT complete until verified.** Code that "looks right" in your head may not render correctly in the browser.

**The iteration loop:**
1. Deploy and clear caches
2. **Run assertions** — fix any failures, re-deploy, repeat until green
3. **Take screenshots** — only after all assertions pass
4. **Read screenshots** using the Read tool to visually inspect
5. If screenshots reveal aesthetic issues not caught by assertions: fix, re-deploy, go to step 2
6. Present final verified screenshots to the user as proof of work

**When presenting to the user**, always show screenshots inline so the user can see exactly what the deployed result looks like. Never describe what it "should" look like — show what it actually looks like.

**What assertions CAN'T catch** (screenshots still needed for):
- Overall visual composition and aesthetics
- Font rendering and antialiasing
- Gradient/shadow rendering
- Subtle alignment from unexpected CSS cascading
- "Does this look good?" subjective assessment

> *Assertions are the diagnosis. Screenshots are the receipt.*

---

## UI Component Registry

### Global Components (shared across ALL pages)

#### Page Hero (`pages.css`)
```
.page-hero              — Container with gradient background
.page-hero__inner       — Centered flex wrapper (max-width: 720px)
.page-hero__eyebrow     — Small uppercase label above title
.page-hero__title       — Large responsive heading (Cormorant Garamond)
.page-hero__subtitle    — Description text below title
.page-hero__count       — Animated count display
.page-hero__count-number — Large sage-colored number
.page-hero__count-label  — Uppercase label below number
```
**Used by**: Shop, Learn, Testimonies, Our Story, Contact, Wholesale
**NEVER create page-specific hero classes.** Use `.page-hero` from `pages.css`.

#### Buttons (`components.css`)
```
.btn                — Base button (flex, uppercase, transition)
.btn--primary       — Dark background
.btn--secondary     — Transparent with border
.btn--coral / .btn--sage — Color variants
.btn--sm / .btn--lg — Size variants
```

#### Product Card (`components.css`)
```
.product-card           — Card with hover lift effect
.product-card__image    — 1:1 aspect ratio container
.product-card__content  — Text area with gradient bg
.product-card__category — Small uppercase label
.product-card__title    — Product name link
.product-card__description — 2-line clamped excerpt
.product-card__price    — Price display
```

#### Badges (`components.css`)
```
.badge / .badge--sage / .badge--coral / .badge--outline
```

### Filter System (shared across Shop, Learn, Testimonies)

#### Filter Bar (`learning-center.css`)
```
.lc-filters             — Sticky container (top: 80px, z-index: 500)
.filter-controls        — Centered flex wrapper
.filter-row             — Horizontal row of filter controls
.lc-search-wrapper      — Search input with border
.lc-search-input        — Text input (transparent bg)
.lc-search-icon         — Magnifying glass SVG
.lc-search-clear        — X button (hidden when empty)
```

#### Custom Select Dropdown (`learning-center.css` + JS)
```
.filter-select-wrapper       — Container (.open class toggles visibility)
.filter-select               — Native <select> (hidden by JS)
.filter-btn                  — Styled trigger button
.filter-btn__label           — Text label inside button
.filter-select-arrow         — Chevron icon (rotates on open)
.filter-dropdown             — Floating list (animated)
.filter-dropdown__item       — Item with hover/selected underline
```

**CRITICAL**: Every `<select>` MUST be initialized with `initCustomSelect(selectEl)`:
```javascript
var customSetValue = initCustomSelect(mySelectElement).setValue;
customSetValue('some-value'); // programmatic update
```
The function lives in each page's JS (shop.js, learning-center.js, testimonies.js). It returns `{ setValue }`. **NEVER leave a raw native `<select>` unstyled.**

Note: `initCustomSelect()` is currently duplicated across three JS files. If you modify it, update all three. This is known tech debt.

#### Load More & No Results (`learning-center.css`)
```
.load-more-wrapper / .load-more-btn / .load-more-btn[hidden]
.lc-no-results / .lc-no-results.visible
.lc-no-results__icon / .lc-no-results__title / .lc-no-results__message
.lc-reset-btn
```

### Animation Classes
```
.animate-on-scroll          — Initial (opacity: 0, translateY: 40px)
.animate-on-scroll.visible  — Revealed (opacity: 1, translateY: 0)
.filtering-out              — Fade out + scale (300ms)
.filtering-in               — Stagger fadeIn (--filter-index CSS var)
```

### CSS Design Tokens (`:root` in `style.css`)

```
Colors:    --color-coral(-light/-dark), --color-sage(-light/-dark),
           --color-cream(-dark), --color-warm-white, --color-text(-light/-muted),
           --color-border, --color-shadow, --color-white
Fonts:     --font-display (Cormorant Garamond), --font-sans (Inter), --font-body
Spacing:   --space-xs(.5rem) / sm(1rem) / md(1.5rem) / lg(2.5rem) / xl(4rem) / 2xl(6rem)
Motion:    --transition-fast(.2s) / medium(.4s) / slow(.6s) / base(=fast)
Layout:    --max-width(1400px), --header-height(110px), --radius-pill(9999px), --radius-card
```

### Responsive Breakpoints

| Width | Context | Layout Changes |
|-------|---------|----------------|
| 1200px | Large tablet | 4→3 column product/article grids |
| 1024px | Tablet | Filter bars stack vertically, 2-column grids |
| 860px | Small tablet | Shop grid 2-column |
| 768px | Mobile landscape | Header collapses to hamburger |
| 640px | Mobile | 1-column grids, stacked hero |
| 480px | Small mobile | Reduced padding, smaller type |

---

## Server & Deployment

**Theme path on server:**
```
***REDACTED_SERVER_PATH***
```

**SSH:** `ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP***`

**Deploy via SCP:**
```bash
scp -i ~/.ssh/***REDACTED_KEY_NAME*** local-file \
  root@***REDACTED_IP***:***REDACTED_SERVER_PATH***path/to/file
```

**Clear ALL caches after every deploy:**
```bash
docker exec valkey valkey-cli FLUSHALL
docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'
```

---

## CSS File Map

| File | Scope | Key Components |
|------|-------|---------------|
| `style.css` | Global | Design tokens (`:root` variables) |
| `base.css` | Global | Typography, lists, links, utilities |
| `components.css` | Global | Buttons, cards, forms, badges |
| `pages.css` | Global | `.page-hero`, page-specific sections |
| `header.css` | Global | Navigation, sticky header |
| `footer.css` | Global | Footer grid, links |
| `animations.css` | Global | Keyframes, scroll reveal |
| `home.css` | Home only | Hero, values, story sections |
| `learning-center.css` | Shared | Filter bar, dropdowns, article cards, load more, no results |
| `shop.css` | Shop only | Product grid, object-fit override, foliage z-index |
| `product.css` | Product only | Single product WooCommerce overrides |
| `testimonies.css` | Testimonies | Rating display, masonry grid |
| `singles.css` | Singles only | Single post/article layout |

---

## WooCommerce Integration

For WooCommerce-specific issues (term cache corruption, visibility taxonomy, template detection, category ordering), read [woocommerce-gotchas.md](references/woocommerce-gotchas.md).

**Quick reference — most common pitfalls:**
- Use `wp_get_object_terms()` not `get_the_terms()` for migrated product categories
- Use `is_shop()` alongside `is_page_template()` for shop page detection
- Always flush Valkey + FastCGI after any product/category changes
- Category order is explicit in `template-shop.php` and `footer.php` — keep in sync with header nav menu
