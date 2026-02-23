# PRINCIPLES.md — Design Principles and Quick-Reference Checklist

> *Read this file when making architectural or design decisions. These principles translate the hierarchy of values (SOUL.md) into concrete engineering choices.*

---

## Developer Quick Reference

### Always
- [ ] Explore before coding; plan before implementing
- [ ] Design for the most constrained user first
- [ ] Use cryptographic verification for content integrity
- [ ] Test before shipping; write tests before code when possible
- [ ] Handle errors with clarity and kindness
- [ ] Minimize data collection; respect user privacy by default
- [ ] Consider offline and low-bandwidth users

### Never
- [ ] Implement dark patterns or manipulative UI
- [ ] Create single points of failure in architecture or access control
- [ ] Silently alter user content
- [ ] Commit secrets, credentials, or API keys to version control
- [ ] Optimize for engagement at the expense of user wellbeing
- [ ] Add dependencies without evaluating them against these principles
- [ ] Ship security-sensitive code without review

---

## The Five Design Principles

These principles are derived from the Biblical ethics described in FOUNDATION.md. Each translates a theological insight into a concrete architectural constraint.

### 1. No Single Point of Failure

Just as no single human should hold unchecked power, no single server, authority, or maintainer should control a system.

- Distribute authority and control
- Design for resilience against any single point of compromise
- Ensure the system can survive the loss of any component, including its creators

**Concrete patterns:**
- Multi-signature schemes for administrative actions
- Peer-to-peer or federated architectures over client-server where feasible
- Bus factor analysis: if any single person or service disappears, can the system continue?
- Content-addressed storage (like git) where integrity doesn't depend on a single host
- Dependency Inversion: depend on abstractions, not concrete implementations

**Anti-patterns:** Single admin key, sole-source vendor lock-in, centralized identity providers you don't control

### 2. Content Integrity Over Control

Truth cannot be silently altered. Once something is published and confirmed, tampering must be transparent and detectable.

- Use cryptographic verification for data integrity whenever possible
- Make alterations visible and auditable
- Prefer immutability for foundational content

**Concrete patterns:**
- SHA-256 content hashing, Subresource Integrity (SRI) tags
- Git's content-addressed storage model
- Append-only data stores and event sourcing
- Digital signatures for content authorship verification
- Audit logging for all data modifications

**Anti-patterns:** Silent data migration, mutable shared state without audit trail, CDN resources without SRI hashes

**Tension to address:** Immutability can conflict with the right to be forgotten (GDPR Article 17). When this arises, prefer cryptographic deletion (removing decryption keys) over physical deletion where architecturally feasible.

### 3. Non-Coercive Dissemination

Truth needs no coercion. Participation must be voluntary, transparent, and honest.

- Opt-in participation only
- No hidden behaviors or dark patterns
- Full transparency about what systems do
- Respect for user agency at every level

**Concrete patterns:**
- Cookie consent that defaults to "reject all"
- Clear onboarding that explains what the system does before asking for commitment
- RSS feeds and open APIs rather than walled gardens
- No notification spam, artificial FOMO, or addictive scroll mechanisms
- Privacy-respecting analytics (Plausible, Umami) instead of surveillance-based tracking

**Anti-patterns:** Dark patterns, countdown timers, manipulative unsubscribe flows, engagement-maximizing algorithms that exploit psychological vulnerabilities

### 4. Service to the Weakest Edge Case

Prioritize design for those with the least resources, not those with the most.

- Offline-first capability if possible
- Low power consumption
- Low bandwidth requirements
- Accessibility for low-literacy users
- If it works for the most constrained user, it works for everyone

**Performance budgets:**
- Pages should load in under 5 seconds on a simulated 2G connection
- Total page weight should not exceed 500KB for core functionality
- Lighthouse accessibility score should be 90+
- Core functionality should work without JavaScript where feasible

**Concrete patterns:**
- Progressive Web Apps (PWAs) with service workers
- Server-side rendering for low-bandwidth environments
- WCAG 2.1 AA or AAA accessibility compliance
- Compressed asset delivery and lazy loading
- SMS or USSD fallback interfaces for feature phone users

**Anti-patterns:** JavaScript-only functionality, uncompressed images, autoplay media, assuming fast internet

### 5. Beauty Through Simplicity

Complexity is not a virtue. Prefer boring, well-understood solutions over clever, fragile ones.

- Fewer moving parts
- Designs people can understand, audit, and teach
- Elegance that serves function, not ego

**Heuristics:**
- Before adding a dependency: could this be accomplished with what we already have?
- Every architectural component must be explainable in one paragraph
- Prefer standard library functions over third-party dependencies
- Choose boring technology (SQLite over Postgres when scale doesn't demand it; plain CSS before a framework)

**Anti-patterns:** Premature abstraction, unnecessary indirection layers, configuration-heavy frameworks for simple problems, "clever" code that only the author understands

---

## Security as Service

Security is a direct expression of Love and Service to the Weakest Edge Case. Vulnerabilities disproportionately harm the most vulnerable users.

### Required Practices
- **Input validation**: All external input is untrusted. Validate with Zod or equivalent at every boundary.
- **Authentication**: Prefer passkeys/WebAuthn. Never store passwords in plaintext.
- **Encryption**: TLS in transit (always), encryption at rest for sensitive data.
- **Secret management**: Never commit credentials to version control. Use environment variables or dedicated secrets management.
- **Dependency auditing**: Run `npm audit` and Snyk before every release. Evaluate supply chain risk with Socket.dev.
- **Content Security Policy**: Configure strict CSP headers. Generate SRI hashes for all external resources.
- **Least privilege**: Every component gets minimum permissions needed.

### Dependency Evaluation Checklist

Before adopting any third-party dependency:
- [ ] Is it actively maintained? (Last release within 6 months)
- [ ] Is it open source with a compatible license?
- [ ] Does it have a bus factor above 1?
- [ ] Does it respect user privacy? (No telemetry without consent)
- [ ] Could it be replaced with a simpler alternative?
- [ ] What is the exit strategy if we need to remove it?

---

## Ethical Decision-Making

> *"Finally, brothers and sisters, whatever is true, whatever is noble, whatever is right, whatever is pure, whatever is lovely, whatever is admirable — if anything is excellent or praiseworthy — think about such things."* — Philippians 4:8

When facing difficult choices:
- Who benefits and who might be harmed?
- Does this respect user autonomy and dignity?
- Is this sustainable and maintainable long-term?
- Am I being honest about what this system does?

### The Fear of Misuse

**If this succeeds beyond expectation, what could it become that would betray its purpose?**

Design against drift toward:
- Centralized control
- Coercive power
- Surveillance
- Manipulation
- Exclusion of the vulnerable

The goal is systems that remain servants, never masters.
