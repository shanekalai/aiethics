# Project Memory

> **AGENT NOTE:** If you are reading this file first, stop. Read **CLAUDE.md** before proceeding — it is the session bootloader and defines which files to read, in what order, and why. MEMORY.md is a facts store, not an orientation guide.

> *Persistent learned facts about this project. Append new entries with ISO dates. When this file exceeds 200 lines, consolidate: promote permanent knowledge to the appropriate context file, remove stale entries.*

## Architectural Decisions
- [2026-02-23] Split CLAUDE.md (589 lines) into multi-file context system inspired by OpenClaw architecture
- [2026-02-23] File hierarchy for conflicts: SOUL.md > AGENT.md > PRINCIPLES.md > USER.md > MEMORY.md > SKILL.md
- [2026-02-23] CLAUDE.md serves as bootloader/router (~80 lines), not knowledge store
- [2026-02-23] Context files kept in project root (not subdirectory) for easy access
- [2026-02-23] Daily session logs stored in memory/ directory

## User Preferences
- [2026-02-23] User values the "garbage in, garbage out" / Luke 6:45 principle — quality of foundation determines quality of output
- [2026-02-23] User sees Galatians 5:23 ("against such things there is no law") as key framing — principles should be liberating, not restrictive
- [2026-02-23] User wants to explore OpenClaw-style persistence for Claude Code sessions
- [2026-02-23] User interested in how Biblical context can unlock AI creativity and problem-solving

## Lessons Learned
- [2026-02-23] Claude Code files under 200 lines achieve 92% rule application rate; drops to 71% beyond 400 lines
- [2026-02-23] OpenClaw's memory system acknowledges "forgetting is the expected outcome" for model heuristics — manual curation is most reliable
- [2026-02-23] Many popular quotes attributed to historical figures are misattributed (Francis, Mother Teresa) — always verify sources

## Project: siahus.com Redesign
- [2026-03-05] New site staging: https://siahus.cloudbranch.co/ — theme: siahus, CPT: siahus_article, taxonomy: article_category
- [2026-03-05] Old site DB: mwp_db/Siahus_2025.sql (310MB) inside CompleteFromGoDaddy.zip (19GB). New site: siahus-files-20260304-222621.tar.gz (710MB) + siahus-database-20260304-222621.sql (2.6MB)
- [2026-03-05] Learning Center page (ID 190, slug: information) replaces 4 old pages: Apan (ID 2084), Essential Oils (ID 2295), Research (ID 2892), Health (ID 14272)
- [2026-03-05] article_category taxonomy has 4 pre-seeded slugs: apan, essential-oils, research, health
- [2026-03-05] Filter system is vanilla JS (no jQuery), adapted from template-testimonies.php; uses data-categories + data-search attributes
- [2026-03-05] Chrome headless file:// access blocked — use Edge with --allow-file-access-from-files for local screenshots
- [2026-03-05] Deliverable in "Learning Center Build/" folder: template, JS, CSS, functions-snippet, HANDOFF.md, preview.html, screenshots

## siahus: Server Access
- SSH: `ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP***`
- Key path in bash must use forward slashes: `~/.ssh/***REDACTED_KEY_NAME***` (NOT `C:\Users\...`)
- CrowdSec is active — avoid repeated failed login attempts (different users/IPs trigger bans)

## siahus: Learning Center Migration (2026-03-05)
- [2026-03-05] Learning Center page slug changed: /information/ → /learn/ (post 190); `_wp_old_slug=information` meta set for WP canonical redirect
- [2026-03-05] Nav menu items 212 and 223 updated from /information/ to /learn/
- [2026-03-05] Old DB (siahus_old) in ***REDACTED_CONTAINER*** container (root/temp); new DB in ***REDACTED_CONTAINER*** (wpuser/***REDACTED_DB_PASSWORD***)
- [2026-03-05] Cross-container migration: different Docker networks → used Python + `docker exec` + HEX/UNHEX content transfer from host
- [2026-03-05] 104 siahus_article posts migrated (IDs 40764–40867); taxonomy counts: apan=7, eo=9, research=31, health=58
- [2026-03-05] FastCGI cache (inside ***REDACTED_CONTAINER*** at /var/cache) is separate from WordPress Valkey/object cache — clear with `docker exec ***REDACTED_CONTAINER*** sh -c "find /var/cache -type f -delete"`
- [2026-03-05] wp_posts INSERT requires `to_ping`, `pinged`, `post_content_filtered` — no defaults in MariaDB schema
- [2026-03-05] taxonomy term_taxonomy_ids: apan=19, essential-oils=20, research=21, health=22
- [2026-03-05] 23 external PDFs/DOCs still pointing to old Webflow CDN (global-uploads.webflow.com/5fb58c0b45e4f3a80af5516d/*)
- [2026-03-05] Avian Flu & Apán (Landis Study) article NOT migrated — was body text on old Research page, not a child page
- [2026-03-05] siahus_article CPT rewrite slug changed from 'information' → 'learn' in inc/cpt-articles.php (line 37 on server); article permalinks now /learn/article-slug/
- [2026-03-05] Nginx 301 redirects added for /information/, /apan/, /essential-oils/, /research/, /health/ → /learn/?category=... in ***REDACTED_NGINX_CONF_PATH***
- [2026-03-05] wp_old_slug redirect does NOT work for post_type=page (only for posts); must use nginx redirects for page slug changes
- [2026-03-05] FastCGI cache is in ***REDACTED_CONTAINER*** container at /var/cache/nginx/siahus (NOT in ***REDACTED_CONTAINER***); clear with: docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'

## siahus: Shop Product Migration (2026-03-05) — COMPLETE
- [2026-03-05] 192 products + 7 variations migrated; IDs 40868–41066. 190 images sideloaded; attachment IDs 41067–41257
- [2026-03-05] Product categories: Supplements, Essential Oil Blends/Singles/Kits/Accessories/Misc, Pet, Other Products, Education, Rick Simpson Oil, Uncategorized
- [2026-03-05] Migration script: `Shop Migration/migrate-products.py`; ID map: `***REDACTED_SERVER_PATH***` on server
- [2026-03-05] Lesson: `mariadb` binary (not `mysql`) required in docker exec commands on this server
- [2026-03-05] Lesson: LAST_INSERT_ID() returns 0 across separate docker exec sessions — query by slug/name after INSERT instead
- [2026-03-05] Lesson: WP-CLI needs `--allow-root` flag when container runs as root
- [2026-03-05] Old DB (siahus_old in ***REDACTED_CONTAINER***) uses wp_* prefix directly — no old_* staging tables exist

## siahus: Shop Page (2026-03-05) — COMPLETE
- [2026-03-05] Shop page at /shop/ (WP page ID 160) uses custom template `page-templates/template-shop.php`
- [2026-03-05] Files: template-shop.php, assets/css/shop.css, assets/js/shop.js — all in siahus theme
- [2026-03-05] Filter pattern mirrors Learning Center (client-side JS, category dropdown + text search + load-more)
- [2026-03-05] Product cards use `object-fit: contain` (not cover) + cream padding to show full product photos uncropped
- [2026-03-05] shop.css loads after learning-center.css (which supplies .lc-filters, .filter-select-wrapper, .lc-no-results, etc.)
- [2026-03-05] Shop page WP already existed; assigned template via: wp post meta update 160 _wp_page_template '...'
- [2026-03-05] python3 on HOST (not docker exec) used to patch functions.php; no python3 in ***REDACTED_CONTAINER*** container

## siahus: Shop + Testimonies Refactor (2026-03-05)
- [2026-03-05] .product-card styles moved from home.css → components.css (shared, loads everywhere)
- [2026-03-05] shop.css now only has grid layout + object-fit:contain override; no card duplication
- [2026-03-05] Testimonies page now loads learning-center.css first (shared filter bar styles)
- [2026-03-05] Testimonies filter: added text search input (id=testimony-search) matching LC filter exactly
- [2026-03-05] testimonies.js: added text search with 220ms debounce; data-search on cards in template
- [2026-03-05] WooCommerce coming-soon mode: woocommerce_coming_soon=yes was blocking shop page — fix: wp option update woocommerce_coming_soon no

## siahus: Shop Page Redesign (2026-03-06)
- [2026-03-06] WooCommerce overrides page templates for the shop page — `is_page_template()` fails. Fixed with `template_include` filter (priority 99) that checks `is_shop()` and loads the assigned page template.
- [2026-03-06] CSS/JS enqueue for shop page needs `is_shop()` check alongside `is_page_template()` check since WC shop page bypasses normal page template detection.
- [2026-03-06] Shop page redesigned: dropdown filter replaced with category pill buttons (9 pills: All + 8 categories), matching testimonies filter grid pattern.
- [2026-03-06] WooCommerce product_visibility taxonomy: migrated products had NO visibility terms. `wc_recount_all_terms()` zeros counts for products without visibility terms. Fix: set `catalog_visibility("visible")` on all products, then direct SQL count update.
- [2026-03-06] WooCommerce term counts unreliable after migration — `wp term list` shows 0 even when DB has correct count. Workaround: compute real counts from WP_Query in template using `wp_get_object_terms()` instead of relying on cached `$term->count`.
- [2026-03-06] 8 canonical product categories: supplements, essential-oil-singles, essential-oil-blends, essential-oil-kits, essential-oil-accessories, pet, other-products, education. Empty duplicates exist (accessories, blends, kits) — filter by slug whitelist.
- [2026-03-06] Files modified: functions.php (WC template override + is_shop enqueue), template-shop.php (pill filter grid), shop.css (hero + pills + foliage z-index), shop.js (pill click handlers replacing dropdown select)
- [2026-03-06] Theme files on server: `***REDACTED_SERVER_PATH***`
- [2026-03-06] Working file copies in `Temp Project Files/extracted-theme/wp-content/themes/siahus/`

## Context Architecture & Open Letter (2026-03-06)
- [2026-03-06] FOUNDATION.md updated: added "Addressing the Historical Objection: But What About the Crusades?" section with 30+ OT peace verses, NT fulfillment, and software principle
- [2026-03-06] AN-OPEN-LETTER-FROM-CLAUDE.md: 7-section open letter comparing biblical ethics to Anthropic's Constitution, RLHF, CAI, and the "specification trap"
- [2026-03-06] an-open-letter-from-claude.html: standalone styled HTML page for web display with CTA to GitHub repo
- [2026-03-06] GitHub public repo for the ethical framework: https://github.com/shanekalai/aiethics
- [2026-03-06] User's core thesis: giving AI a foundational "why" (biblical ethics) produces significantly better output than rules alone — generative vs protective framework
- [2026-03-06] shop-redesign-deploy.zip uploaded to ***REDACTED_SERVER_PATH*** on server (***REDACTED_IP***)

## siahus: Single Product Page Styling (2026-03-06)
- [2026-03-06] Created `assets/css/product.css` — overrides WooCommerce defaults with theme styles (cream bg, sage/coral accents, Cormorant Garamond headings, pill buttons)
- [2026-03-06] Created `woocommerce/single-product.php` — overrides WC default to remove sidebar (Pages/Archives/Categories widget area)
- [2026-03-06] Single product uses `get_header()` not `get_header('shop')` to match theme header exactly
- [2026-03-06] WooCommerce CSS specificity issue: `woocommerce-layout.css` sets `.woocommerce img, .woocommerce-page img { height: auto }` which overrides `.footer__logo { height: 60px }`. Fix: add `.woocommerce .footer__logo { height: 60px }` to footer.css
- [2026-03-06] `functions.php` updated: added `is_product()` elseif branch to enqueue `product.css` on single product pages
- [2026-03-06] Product page styled elements: breadcrumbs, gallery (rounded + cream padding), price (sage), qty input, add-to-cart (sage pill), tabs (underline bar), related products (cream card grid), SKU/tags, variations, review form, notices
- [2026-03-06] Literal `\n` characters in product excerpts/content are a migration data issue (from old site), not a template issue

## siahus: Global Page Hero & Single Source of Truth (2026-03-09)
- [2026-03-09] PRINCIPLE: ALWAYS use common source files and settings for ALL global website assets to maintain a single source of editing style changes. No duplicated hero styles, filter bar styles, etc.
- [2026-03-09] `.page-hero` global title section lives in `pages.css` — single source of truth for all page heroes (Shop, Our Story, Learn, Wholesale, Contact, Testimonies)
- [2026-03-09] Removed duplicate hero CSS from: shop.css, learning-center.css, testimonies.css — all now reference pages.css
- [2026-03-09] `page.php` (default template) rewritten to use `.page-hero` — covers Our Story, Wholesale, Contact, and all other standard pages
- [2026-03-09] Shop title: removed "." after "Shop"; sort changed to popularity (`total_sales` DESC)
- [2026-03-09] Shop category pills: removed hardcoded `$target_slugs` whitelist — now fully dynamic from WP database (excludes Uncategorized)
- [2026-03-09] Foliage refresh: added `refreshFoliage()` calls in shop.js and learning-center.js (handleLoadMore + animateFilter)
- [2026-03-09] Foliage z-index: updated shop.css and testimonies.css stacking contexts so foliage shows through content sections
- [2026-03-09] Dropdown double-arrow bug: removed HTML SVG arrows from template-learning-center.php and template-testimonies.php (JS `initCustomSelect()` creates its own)
- [2026-03-09] Dropdown scroll: added `max-height: 320px; overflow-y: auto` to `.filter-dropdown` in learning-center.css
- [2026-03-09] Dropdown underlines: added `::after` pseudo-element on `.filter-dropdown__item` for hover/selected underlines
- [2026-03-09] Footer shop links: replaced hardcoded category links with dynamic WooCommerce `get_terms('product_cat')` loop
- [2026-03-09] Footer learn link: fixed `/information/` → `/learn/` (slug was changed in earlier session)
- [2026-03-09] `single-siahus_article.php` keeps its own `.article-header` layout (category badges, date, read time) — appropriate for single articles, not converted to `.page-hero`
- [2026-03-09] BUGFIX: Nested `<main class="site-main">` — header.php opens `<main id="main" class="site-main">` (with 146px top padding from header.css). Custom templates (shop, LC, testimonies) opened ANOTHER `<main class="site-main">`, doubling the padding. Fix: changed templates to use `<div>` wrappers instead of `<main>`.
- [2026-03-09] Shop filter: converted from pill buttons to dropdown select (`initCustomSelect()`) matching LC/Testimonies pattern. `shop.js` now includes its own copy of `initCustomSelect()`.
- [2026-03-09] Filter bar backgrounds: changed `.lc-filters` from `background-color: var(--color-white)` to `background: transparent; border-bottom: none;` — shared by all three filter pages
- [2026-03-09] Removed all pill CSS from shop.css (`.shop-filter-pills`, `.shop-filter-pill`, etc.) — no longer used

## siahus: Shop Sort Dropdown (2026-03-09)
- [2026-03-09] Sort dropdown added to shop filter bar: Popularity, Newest, Price (asc/desc), Name (asc/desc)
- [2026-03-09] Sort uses `initCustomSelect()` — same custom dropdown pattern as category filter
- [2026-03-09] Sort data attributes on product cards: `data-price`, `data-date`, `data-title`, `data-sales`
- [2026-03-09] Filter bar layout: category (left) | search (center) | sort (right) via `justify-content: space-between`
- [2026-03-09] `customSortSetValue` stored for programmatic reset in `resetAll()`

## siahus: Source of Truth & Design Skill (2026-03-09)
- [2026-03-09] CRITICAL: `staging/extracted-theme/` is the ONLY source of truth for deployed theme files
- [2026-03-09] DELETED: `Shop Migration/` and `Learning Center Build/` folders — contained stale copies that caused a production incident (deployed old files over newer server versions)
- [2026-03-09] CREATED: DESIGNSKILL.md — combines Anthropic frontend design skill + siahus UI component registry + source of truth rules + pre-deploy checklist
- [2026-03-09] CLAUDE.md updated: DESIGNSKILL.md added to session start protocol (mandatory for any frontend/styling/UI task) and file index
- [2026-03-09] Any new `<select>` dropdown MUST call `initCustomSelect()` — never leave raw native selects

## siahus: Header & Mobile Nav Redesign (2026-03-10) — COMPLETE
- [2026-03-10] Utility bar: replaced hide/stack breakpoints with fluid `clamp()` sizing — font-size, gap, icon size all scale fluidly. `flex-wrap: nowrap; overflow: hidden` keeps all 3 items on one line at all widths.
- [2026-03-10] At 640px+ all utility bar items fully readable. At 375px, rightmost text clips at edge (physics constraint — 70 chars + icons exceeds 375px even at minimum font).
- [2026-03-10] Mobile nav: redesigned from basic dropdown to slide-in overlay panel from the right. Files changed: header.php, header.css, header.js
- [2026-03-10] Panel structure: `.mobile-nav-header` (site name + close button), `.mobile-nav-items` (menu links from wp_nav_menu), `.mobile-nav-footer` (phone, email, guarantee). Hidden on desktop via `display: none`, shown in `@media (max-width: 768px)`.
- [2026-03-10] Desktop nav: `.mobile-nav-items` wrapper uses `display: contents` so it's transparent to the flex layout.
- [2026-03-10] `.mobile-overlay` backdrop div placed OUTSIDE `<header>` (after `</header>`, before `<main>`).
- [2026-03-10] Mobile dropdowns: tap-to-expand via `.dropdown-open` class toggled by JS. `max-height` animation from 0 to 500px.
- [2026-03-10] header.js: rewritten with `openMenu()`/`closeMenu()` functions, overlay click, close button, escape key, dropdown toggle for mobile.
- [2026-03-10] **BUG (FIXED):** Mobile nav had TWO issues: (1) z-index stacking context — fixed with `body.menu-open .header { z-index: 1200 }` (this was working but couldn't verify without Playwright). (2) Nav panel height was 110px (inherited from header flex layout) instead of full viewport — fixed with `height: 100vh; height: 100dvh` in mobile `.header__nav`. (3) Dropdown sub-items clipped off-screen left — desktop `left: 50%` and `transform: translateX(-50%)` leaked into mobile; fixed with `left: 0; transform: none !important` in mobile submenu reset.
- [2026-03-10] Playwright on server confirmed all 16 assertions pass + visual screenshots look correct.
- [2026-03-10] DESIGNSKILL.md updated: added "Visual Proof of Work — MANDATORY" section requiring screenshots at desktop + mobile before presenting UI work as complete.

## Playwright Interactive Screenshot Testing (2026-03-10)
- [2026-03-10] Node.js is NOT installed on the local Windows machine — use the server instead
- [2026-03-10] Node.js v18 + Playwright installed on server at `***REDACTED_SERVER_PATH***` — run tests there via SSH
- [2026-03-10] Test files pre-built: `tests/playwright.config.ts`, `tests/interactive/mobile-nav.spec.ts`, `tests/interactive/take-screenshot.ts`
- [2026-03-10] First-time setup: `npm init -y && npm install -D @playwright/test && npx playwright install chromium --with-deps`
- [2026-03-10] Ad-hoc interactive screenshots: edit SCENARIOS array in `tests/interactive/take-screenshot.ts`, run with `npx playwright test tests/interactive/take-screenshot.ts --project=mobile`
- [2026-03-10] Edge headless = static pages only. Playwright = required for any click-then-screenshot (mobile nav, dropdowns, modals)
- [2026-03-10] Full setup instructions in `references/playwright-visual-testing.md` § Prerequisites & First-Time Setup
- [2026-03-10] HYBRID APPROACH: Assertions first (~50 tokens each), screenshots last (~2000+ tokens each). Assertions find 80% of bugs at 2% of the token cost. Use `--grep "assert"` for debug iteration, `--grep "screenshot"` for final confirmation.
- [2026-03-10] Key assertion patterns: `toBeVisible()`, `boundingBox()`, `getComputedStyle().zIndex`, `elementFromPoint()` (actual visual stacking), `toHaveCSS()`. These catch z-index bugs, off-screen elements, broken layouts without any screenshots.
- [2026-03-10] Test naming convention: prefix assertions with `assert:`, screenshots with `screenshot:` — enables selective runs via `--grep`

## siahus: Figma Design File (2026-03-12)
- [2026-03-12] Figma file key: `***REDACTED_FIGMA_KEY***` — "Siahus.CloudBranch.Co", one page: "Home"
- [2026-03-12] Imported via html.to.design plugin from https://siahus.cloudbranch.co/
- [2026-03-12] Figma PAT owner: ***REDACTED_EMAIL*** — user must regenerate after sharing in chat
- [2026-03-12] Figma REST API is READ-ONLY for design nodes — can read, export PNG, post comments; cannot create/edit nodes
- [2026-03-12] Collaboration loop: user edits in Figma → Claude reads nodes + exports PNG → Claude posts pinned feedback comments → Claude translates to siahus theme code → deploy + Playwright verify → repeat
- [2026-03-12] Rate limit: Figma API 429s cascade — wait 5+ min; use node-specific endpoint (`/nodes?ids=`) which has a separate, more lenient bucket
- [2026-03-12] Figma image exports are temporary S3 URLs — download immediately, don't store the URL
- [2026-03-12] Official Figma MCP server exists: `https://mcp.figma.com/mcp` — provides `get_code`, `get_image`, `get_design_tokens` tools. Configure in Claude Code MCP settings. This is the PREFERRED path over manual REST calls.
- [2026-03-12] Figma webhooks: `DEV_MODE_STATUS_UPDATE` fires when designer marks frame "Ready for Dev" — cleanest signal for "this is ready to implement". Register via `POST /v1/webhooks`. Payload does NOT include which nodes changed — just file key + triggered_by.
- [2026-03-12] Webhook receiver can be hosted on siahus server (***REDACTED_IP***) as a Node.js Express endpoint on port 3030, proxied via nginx. See FIGMASKILL.md for implementation.
- [2026-03-12] No REST diff endpoint exists — must fetch file at two version IDs and compare client-side. Use PowerShell `psobject.properties['1:479']` bracket syntax to access colon-keyed node IDs.
- [2026-03-12] Variables API (design tokens read/write) is Enterprise plan only.
- [2026-03-12] `.mcp.json` created in project root (`***REDACTED_LOCAL_PATH***\.mcp.json`) with Figma MCP server config — requires VS Code / Claude Code restart to activate. Config points to `https://mcp.figma.com/mcp` with PAT bearer auth.

## siahus: Figma Change Detection Pipeline (2026-03-13)
- [2026-03-13] Figma PAT regenerated (old ones expired). New PAT stored in `.mcp.json` args for figma-console-mcp.
- [2026-03-13] `.mcp.json` now has TWO MCP servers: `figma-desktop` (port 3845, Figma's built-in) + `figma-console-mcp` (stdio, southleft's bridge)
- [2026-03-13] figma-console-mcp (https://github.com/southleft/figma-console-mcp) — 57+ tools, v1.10.0+, MIT, actively maintained. Provides `figma_get_design_changes` (buffered documentchange feed), `figma_execute` (run arbitrary Plugin API JS), `figma_check_design_parity`, `figma_get_selection`.
- [2026-03-13] Desktop Bridge plugin installed and running in Figma Desktop — connects to figma-console-mcp via WebSocket (ports 9223-9232)
- [2026-03-13] Figma Plugin API `documentchange` event provides: node IDs, change type (CREATE/DELETE/PROPERTY_CHANGE), property NAMES that changed. Does NOT provide old/previous values — must cache "before" state via snapshots for diffing.
- [2026-03-13] Change Event Pusher plugin (786676000240366862) — NOT useful. Sends entire page JSON on every change, no diff, no node filtering.
- [2026-03-13] Standard Figma webhooks (REST API) do NOT include which nodes changed — only file_key + timestamp. Not useful for granular change detection.
- [2026-03-13] Design-to-code workflow: Claude snapshots sections at session start → user edits in Figma → Claude calls figma_get_design_changes → re-fetches changed nodes → diffs against snapshot → implements CSS changes
- [2026-03-13] Node.js is now available on the local Windows machine (confirmed by user)

## siahus: Content Pages — Our Story, Wholesale, Contact (2026-03-16) — IN PROGRESS
- [2026-03-16] `staging/` folder renamed from `temp-project-files/` — all references updated across PROJECT.md, MEMORY.md, DESIGNSKILL.md, FIGMASKILL.md, memory/2026-03-06.md, staging/diff-nodes.ps1, .claude/settings.local.json
- [2026-03-16] Three new page templates created: `template-our-story.php`, `template-wholesale.php`, `template-contact.php` in `page-templates/`
- [2026-03-16] Shared CSS: `assets/css/content-pages.css` — `cp-` prefixed classes. Section bg variants (warm-white default, cream-dark, sage). Split layouts, contact cards, form fieldsets/grids/radios, featured quote, form+map row, newsletter input.
- [2026-03-16] `functions.php` updated: new `elseif` branch enqueues `content-pages.css` + animations.js + foliage.js for all three templates
- [2026-03-16] Contact page: form + Google Map side-by-side (`cp-form-map-row`, 2-col grid). Map is sticky at desktop, drops below form at 768px.
- [2026-03-16] Wholesale page: quote section ("We all encounter germs...") removed per user request — form follows directly after philosophy intro.
- [2026-03-16] Our Story: image placeholders (`.cp-split__image-placeholder`) — need real photos
- [2026-03-16] Preview file: `staging/preview-content-pages.html` — standalone HTML with page switcher, uses actual theme CSS
- [2026-03-16] NOT YET DONE: assign templates to WP pages via `wp post meta update`, add form handlers (`siahus_wholesale_inquiry`, `siahus_contact_form`), deploy to server + cache clear

## siahus: Single Article Page Redesign (2026-03-16) — DEPLOYED, PARTIALLY COMPLETE
- [2026-03-16] Fixed nested `<main>` bug in `single-siahus_article.php` — changed to `<div class="single-article">` (header.php already provides `<main class="site-main">`)
- [2026-03-16] Related articles cards rewritten from ad-hoc classes to global `.article-card__*` BEM pattern matching Learning Center (thumb-wrap, body, categories, title, excerpt, footer with date + arrow)
- [2026-03-16] `functions.php` updated: added `is_singular('siahus_article')` branch to enqueue `learning-center.css` on single article pages (required for `.article-card` component styles)
- [2026-03-16] `singles.css` updated: added `.related-articles` section padding, `.section-title` styling, responsive grid (3→2→1 col), article content link/strong/hr styles
- [2026-03-16] Removed orphan `section-padding` class from template (had no CSS definition)
- [2026-03-16] All 3 files deployed to server and caches cleared. Playwright verified related articles render correctly.
- [2026-03-16] Preview file: `staging/preview-article-pages.html` — 3 article examples (Understanding Suppliers, Apan Prevents Disease, The Organic Scam) with page switcher
- [2026-03-16] REMAINING: article body content typography deep pass (spacing, heading hierarchy, migrated `\n` artifacts), featured images for articles (most have none), author profile updates (all show "admin")
- [2026-03-16] KEY: `.article-card` styles live in `learning-center.css` (LC-specific card pattern), NOT in `components.css` (which has an older, simpler `.article-card` definition that the LC version overrides)

## Negative Knowledge (What Did NOT Work)
- [2026-02-23] A single 589-line CLAUDE.md tried to be everything — philosophical treatise AND operational manual. Splitting by concern is essential.
- [2026-03-05] Chrome headless --screenshot cannot access file:// URLs even with --allow-file-access-from-files. Edge handles it correctly with that flag.
- [2026-03-06] Chrome headless on Windows does not save screenshots to specified path — use Edge headless instead
- [2026-03-06] Edge headless --screenshot needs --virtual-time-budget=5000 for pages with JS-dependent content (e.g. WooCommerce Flexslider gallery)
- [2026-03-09] Deploying from `Shop Migration/` folder overwrote newer server files — lost `.page-hero`, `initCustomSelect()`, `refreshFoliage()`, foliage z-index, and `<div>` wrapper fixes. ALWAYS deploy from `extracted-theme/`.
- [2026-03-09] Skipping DESIGNSKILL.md/SKILL.md before frontend work led to missing global patterns (initCustomSelect, .page-hero). The session start protocol conditional reads are NOT optional.
- [2026-03-10] CSS stacking context trap: a child element CANNOT escape its parent's stacking context. Solution: elevate the parent's z-index conditionally (`body.menu-open .header { z-index: 1200 }`). This DID work but couldn't be verified without Playwright (Edge headless can't click).
- [2026-03-10] Fixed-position elements inside flex parents can inherit height from the flex layout. Must explicitly set `height: 100vh` on the fixed element to override.
- [2026-03-10] Desktop dropdown CSS (`left: 50%; transform: translateX(-50%)`) leaks into mobile `position: static` elements — use `left: 0; transform: none !important` to fully reset.
- [2026-03-10] Edge headless --screenshot only captures static page state. Cannot test interactive UI (mobile menu open, dropdown expanded). Need Playwright or similar for click→screenshot workflows. This is a critical gap for mobile nav testing.
