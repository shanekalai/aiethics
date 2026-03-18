## Project Overview

**website design for siahus** — WordPress theme development and AI context framework for [siahus.cloudbranch.co](https://siahus.cloudbranch.co/). The repo contains the `siahus` custom WordPress theme, migration scripts, Playwright tests, Figma integration tooling, and a multi-file AI agent context system.

## Architecture

### WordPress Theme (source of truth)
The deployed theme lives at:
```
staging/extracted-theme/wp-content/themes/siahus/
```
**This is the ONLY source of truth for website files.** The /staging/ folder is where all staged edits to the site are built and referenced.  Always assume we are building and testing locally in the staging folder and that implementation will be handled by a separate agent unless otherwise stated.

Theme structure:
- `functions.php` — enqueues, CPT registration, WooCommerce overrides
- `page-templates/` — template-home, template-shop, template-learning-center, template-testimonies
- `assets/css/` — base.css, components.css (shared), pages.css (global `.page-hero`), header.css, footer.css, plus page-specific: shop.css, learning-center.css, testimonies.css, product.css, home.css
- `assets/js/` — header.js, foliage.js, shop.js, learning-center.js, testimonies.js, animations.js
- `inc/` — cpt-articles.php, cpt-testimonials.php, customizer.php, class-siahus-nav-walker.php
- `woocommerce/single-product.php` — WC single product override (removes sidebar)

### CSS architecture
- `components.css` loads globally — shared card styles, filter bar base
- `pages.css` owns the `.page-hero` section used by all pages — single source of truth, never duplicate
- `learning-center.css` supplies `.lc-filters`, `.filter-select-wrapper` etc. — also loaded by shop and testimonies
- All custom `<select>` dropdowns must use `initCustomSelect()` (defined in each page's JS) — never leave raw native selects

### Custom Post Types
- `siahus_article` (slug: `/learn/`) with taxonomy `article_category` (apan, essential-oils, research, health)
- `siahus_testimonial`
- WooCommerce products at `/shop/`

### Server Infrastructure
- **Server:** ***REDACTED_IP*** (Docker-based WordPress)
- **SSH:** `ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP***` (use forward slashes in path, even on Windows)
- **CrowdSec** active — avoid repeated failed SSH attempts
- **Theme path on server:** `***REDACTED_SERVER_PATH***`
- **DB container:** ***REDACTED_CONTAINER*** (MariaDB, use `mariadb` not `mysql` in docker exec)
- **WP container:** ***REDACTED_CONTAINER*** (WP-CLI needs `--allow-root`)

### Cache clearing (two separate systems)
```bash
# WordPress object cache (Valkey/Redis)
docker exec ***REDACTED_CONTAINER*** wp --allow-root cache flush

# FastCGI page cache (in ***REDACTED_CONTAINER*** container, NOT ***REDACTED_CONTAINER***)
docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'
```

### Figma Integration
- Figma file key: `***REDACTED_FIGMA_KEY***`
- Two MCP servers configured in `.vscode/mcp.json`: `figma-desktop` (port 3845) and `figma-console-mcp` (stdio)
- figma-console-mcp provides `figma_get_design_changes`, `figma_execute`, `figma_check_design_parity`
- Design-to-code workflow: snapshot sections → user edits Figma → detect changes → diff → implement

## Common Commands

### Deploy theme files to server
```bash
# From local machine — upload changed file(s)
scp -i ~/.ssh/***REDACTED_KEY_NAME*** \
  "staging/extracted-theme/wp-content/themes/siahus/<file>" \
  root@***REDACTED_IP***:***REDACTED_SERVER_PATH***<file>

# Then clear caches
ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP*** \
  "docker exec ***REDACTED_CONTAINER*** wp --allow-root cache flush && docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'"
```

### WP-CLI on server (via SSH + docker exec)
```bash
docker exec ***REDACTED_CONTAINER*** wp --allow-root <command>
# Examples:
docker exec ***REDACTED_CONTAINER*** wp --allow-root post list --post_type=siahus_article
docker exec ***REDACTED_CONTAINER*** wp --allow-root option update <key> <value>
docker exec ***REDACTED_CONTAINER*** wp --allow-root rewrite flush
```

### Playwright tests (run on server, not local)
```bash
ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP***
cd /root/playwright-tests
npx playwright test tests/interactive/mobile-nav.spec.ts --project=mobile
```
- Prefix assertion tests with `assert:`, screenshot tests with `screenshot:` — run selectively with `--grep`
- Assertions first (~50 tokens), screenshots last (~2000+ tokens) — assertions find 80% of bugs at 2% token cost

### Local screenshots (static pages only)
```bash
# Edge headless — cannot test interactive UI (clicks, menus)
msedge --headless --screenshot="output.png" --window-size=1440,900 --virtual-time-budget=5000 <url>
```

### Migration scripts
- `migrate.py` — cross-container DB migration using Python + `docker exec` + HEX/UNHEX
- `extract_urls.py` — URL extraction utility

## Key Gotchas

- **WooCommerce template override:** `is_page_template()` fails on the WC shop page. Use `template_include` filter (priority 99) checking `is_shop()` instead.
- **WooCommerce term counts:** unreliable after migration. Use `wp_get_object_terms()` in templates instead of `$term->count`.
- **`_wp_old_slug` redirect:** does NOT work for `post_type=page` (only posts). Use nginx 301 redirects for page slug changes.
- **MariaDB wp_posts INSERT:** `to_ping`, `pinged`, `post_content_filtered` have no defaults — must include explicitly.
- **`LAST_INSERT_ID()`** returns 0 across separate `docker exec` sessions — query by slug/name after INSERT instead.
- **Nested `<main>` bug:** `header.php` opens `<main class="site-main">`. Custom templates must use `<div>` wrappers, not another `<main>`.
- **CSS stacking context:** child elements cannot escape parent stacking context. Use conditional parent elevation (e.g., `body.menu-open .header { z-index: 1200 }`).

---