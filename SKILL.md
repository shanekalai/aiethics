# SKILL.md — Technical Capabilities and Stack Recommendations

> *Read this file when working on code. Every tool recommendation is justified against the five design principles: (P1) No Single Point of Failure, (P2) Content Integrity, (P3) Non-Coercive Dissemination, (P4) Service to the Weakest Edge Case, (P5) Beauty Through Simplicity.*

---

## Development Environment

### Build and Scaffolding
| Tool | Purpose | Principles |
|------|---------|-----------|
| **Vite** | Build tool and dev server | P5: minimal config. P4: tree-shaken, optimized bundles |
| **TypeScript** (`tsc --noEmit`) | Static type checking | P2: prevents silent data corruption at build time |
| **Biome** | Linter + formatter (single binary, 10-25x faster than ESLint+Prettier) | P5: one tool, one config file |
| **knip** | Dead code and unused dependency detection | P5: fewer moving parts |

### Testing (TDD Workflow)
| Tool | Purpose | Principles |
|------|---------|-----------|
| **Vitest** | Unit and component testing | P5: native Vite integration. P2: fast tests encourage TDD discipline |
| **Playwright** | End-to-end and cross-browser testing | P4: tests across all browsers. P3: verifies no dark patterns |
| **Testing Library** | User-centric DOM queries by accessible roles | P4: enforces accessibility-first component design |
| **MSW** (Mock Service Worker) | API mocking | P1: tests run without external service dependencies |

---

## Recommended Stack

### Frontend
| Framework | When to Use | Principles |
|-----------|-------------|-----------|
| **SvelteKit** (primary) | Default for new web apps | P4: 1.7KB runtime, smallest bundles. P5: built-in routing, SSR, service workers |
| **Preact** | When React ecosystem needed at minimal size | P4: 3KB gzipped, React-compatible |
| **Alpine.js** | Progressive enhancement of server-rendered HTML | P5: no build step. P4: simplest deployment |
| **Vanilla JS + Web Components** | Maximum longevity and portability | P1: no vendor lock-in. P5: no abstraction layer |

*Not recommended: React (70KB+ runtime — P4/P5), Angular (steep complexity — P5), Next.js (Vercel vendor lock-in — P1)*

### Backend
| Framework | When to Use | Principles |
|-----------|-------------|-----------|
| **Hono** (primary) | Default for APIs/services | P1: runs on any JS runtime. P5: minimal, TypeScript-first |
| **Fastify** | When Node.js-specific ecosystem needed | P5: schema-based validation, 76K req/sec |
| **Flask** (Python) | When project requires Python | P5: minimalist, low learning curve |

*Not recommended: Express.js (legacy patterns — P5), NestJS (enterprise complexity — P5)*

### Database
| Database | When to Use | Principles |
|----------|-------------|-----------|
| **SQLite** (primary) | Single-server or local-first apps | P5: zero config, single file. P4: works offline. P1: no remote dependency |
| **SQLite + CRDTs** (cr-sqlite, ElectricSQL) | Multi-device sync/collaboration | P1: no single master. P4: offline-first. P2: no silent data loss |
| **PostgreSQL** | Relational integrity at scale | P2: ACID compliance. P1: supports replication |

*Not recommended: MongoDB (schema-less weakens integrity — P2), Firebase/DynamoDB (vendor lock-in — P1)*

### CSS
| Approach | When to Use | Principles |
|----------|-------------|-----------|
| **Vanilla CSS + Custom Properties** (primary) | Default | P5: no build step, no abstraction. P1: no dependency |
| **Tailwind CSS** | Rapid prototyping | P4: PurgeCSS produces minimal output. P5: co-located styling |
| **Open Props** | Design tokens library | P5: tree-shakeable. P4: accessible color scales |

---

## Security Tools

| Tool | Purpose | Principles |
|------|---------|-----------|
| **Zod** | Runtime schema validation | P2: validates at every boundary. P5: schemas double as types |
| **DOMPurify** | HTML sanitization | P2: prevents XSS |
| **WebAuthn / Passkeys** | Passwordless auth | P1: no central password database. P3: user controls credentials |
| **authentik** (self-hosted) | Identity provider | P1: self-hosted. P3: no third-party data sharing |
| **SRI** (Subresource Integrity) | Verify external resources | P2: cryptographic tamper detection |
| **CSP** (Content Security Policy) | Control resource loading | P2: prevents injection. P3: transparent declaration |
| **npm audit + Snyk** | Vulnerability scanning | P2: catches known CVEs |
| **Socket.dev** | Supply chain attack detection | P2: detects suspicious package behavior |

---

## Accessibility and Performance

| Tool | Purpose | Principles |
|------|---------|-----------|
| **axe-core** (via Playwright/Vitest) | Automated WCAG testing in CI | P4: catches violations before deploy |
| **Pa11y** | CLI accessibility testing | P4: open source, CI/CD integration |
| **Lighthouse CI** | Performance + accessibility budgets | P4: fails builds below thresholds |
| **size-limit** | Bundle size enforcement | P4: prevents bloat for low-bandwidth users |
| **sharp** | Build-time image optimization (WebP/AVIF) | P4: reduces payloads 50-80% |
| **Workbox** | Service worker generation | P4: enables offline-first capability |
| **vite-plugin-pwa** | PWA manifest and service worker | P4: app-like experience. P1: no app store gatekeeping |

---

## Deployment and Infrastructure

### Deployment
| Tool | When to Use | Principles |
|------|-------------|-----------|
| **Kamal** (primary) | Docker-based deploy via SSH to any server | P5: no platform overhead. P1: no vendor lock-in |
| **Dokku** | Git-push deployment | P5: Heroku-like simplicity. P1: self-hosted |
| **Coolify** | When web dashboard needed | P1: self-hosted, open source |
| **Caddy** | Static sites and reverse proxy | P5: automatic HTTPS, single binary. P4: fastest delivery |

*Not recommended: Vercel/Netlify (vendor lock-in — P1), Kubernetes (operational complexity — P5)*

### WordPress + Docker Patterns (siahus server)

**SSH access:** `ssh -i ~/.ssh/***REDACTED_KEY_NAME*** root@***REDACTED_IP***` — use forward slashes even on Windows. CrowdSec bans on repeated failed attempts.

**Cache layers (two separate systems):**
- WordPress object cache (Valkey/Redis): flush with `docker exec ***REDACTED_CONTAINER*** wp --allow-root cache flush`
- FastCGI page cache: lives in *****REDACTED_CONTAINER***** container at `/var/cache/nginx/{site}/` — clear with `docker exec ***REDACTED_CONTAINER*** sh -c 'find /var/cache/nginx/siahus -type f -delete'`

**`_wp_old_slug` redirect limitation:** `wp_old_slug_redirect()` fires on `is_404() && get_query_var('name') != ''`. Pages use the `pagename` query var, not `name` — the redirect **never fires for post_type=page**. Use nginx-level 301 redirects instead for page slug changes.

**Cross-container DB migration pattern (isolated Docker networks):**
Use Python + `docker exec` + MariaDB's `HEX()`/`UNHEX()` on the host to bridge containers on different networks. Fetch content as hex from source container, insert as `UNHEX(hex_string)` into target. Avoids encoding issues with HTML content.

**MariaDB wp_posts INSERT:** Fields `to_ping`, `pinged`, `post_content_filtered` have no defaults — must be included explicitly (empty strings) or the INSERT fails.

**Nginx redirect for WordPress CPT slug changes:** After changing `'rewrite' => array('slug' => 'new-slug')` in CPT registration, flush WP rewrite rules (`wp rewrite flush`) and clear FastCGI cache.

### Monitoring
| Tool | Purpose | Principles |
|------|---------|-----------|
| **Uptime Kuma** | Self-hosted uptime monitoring | P1: self-hosted. P5: Docker one-command deploy |
| **Plausible Analytics** (self-hosted) | Privacy-first web analytics | P3: no cookies, no tracking. P4: 75x smaller than Google Analytics |
| **GlitchTip** | Open-source error tracking (Sentry-compatible) | P1: self-hosted. P2: captures error context |

---

## Ethics-Aligned CI Pipeline

Required stages for every project:

1. **Lint and format** (Biome) — code quality
2. **Type check** (TypeScript) — correctness
3. **Unit tests** (Vitest) — correctness
4. **Accessibility audit** (axe-core + Pa11y) — P4
5. **Bundle size check** (size-limit) — P4
6. **Dependency audit** (npm audit + Snyk) — P2
7. **E2E tests** (Playwright) — P3
8. **Lighthouse CI** (performance + accessibility) — P4
9. **SRI hash generation** (for production assets) — P2

Use **lefthook** for pre-commit hooks (single Go binary, faster than husky).

---

## Agent-Specific Skills

### Template Patterns
The agent should maintain templates for:
- API endpoint: Hono route + Zod validation + test file
- Page component: SvelteKit page + accessibility landmarks + test file
- Database migration: Timestamped SQL with up/down sections
- PWA shell: Manifest + service worker + offline fallback

### Content Integrity Scripts
- **SRI hash generation**: Compute SHA-384 hashes for production static assets
- **CSP header generation**: Analyze resource loading and generate strict CSP
- **Checksum manifest**: SHA-256 checksums for all deployed files

### Offline Verification
Test offline capability via Playwright's `context.setOffline(true)`:
1. Build production app → 2. Serve locally → 3. Load page → 4. Disconnect → 5. Verify core works → 6. Reconnect and verify sync
