# xtoque — inventory management for small businesses

**Live:** [xtoque.com](https://xtoque.com) · **Status:** in production since 2026 · **Author:** Arthur Isnard (solo)

> This repository is a public case study. The application source code is kept private because the app is in production and holds real customer data. Code walkthroughs are available on request.

<!-- TODO: add 2–3 screenshots in ./screenshots and uncomment
![Inventory count screen](screenshots/inventory.png)
![Admin backoffice](screenshots/backoffice.png)
-->

## The problem

A small business was counting stock on paper and re-typing it into spreadsheets to place supplier orders. xtoque replaces that with a mobile-friendly web app: staff count products per supplier, managers export ready-to-send CSV orders.

## Features

- **Inventory counts** per supplier and product, optimised for mobile use in the stockroom
- **CSV export** of counts for supplier orders
- **Admin backoffice** to manage suppliers, products, teams and members
- **Multi-tenancy**: teams isolate suppliers and members
- **Authentication & roles**: Supabase JWT auth with role-based authorization
- **Catalog templates** published as immutable snapshots
- **GDPR**: personal data export and account deletion

## Tech stack

| Layer | Choice |
|---|---|
| Language | TypeScript (Node.js 24) |
| Web | Express, server-rendered EJS views |
| Database | PostgreSQL (Supabase), raw `pg` adapter |
| Auth | Supabase Auth (JWT) |
| Security | Helmet, rate limiting, server-side authorization |
| Testing | Jest, Supertest, Testcontainers (real PostgreSQL) |
| Observability | Pino structured logging |
| Hosting / CI | Vercel, GitHub Actions |
| Tooling | mise, just |

## Architecture

The code is organised **by domain** (`auth`, `account`, `catalog`, `inventory`, `suppliers`, `members`, `teams`, `profile`), each owning its routes, use cases, repositories and tests — a DDD-lite, ports & adapters approach:

```
HTTP route ──▶ use case ──▶ port (interface) ──▶ adapter (PostgreSQL / Supabase / in-memory)
```

- Use cases depend only on ports, so business logic is tested against in-memory adapters (fast) while adapters are tested against a real PostgreSQL container.
- Dependencies are wired in a single, manual **composition root** — no DI framework.
- Reads always go through a use case, so authorization is enforced in one place.

## Architecture Decision Records

Every significant decision is documented as an ADR (17 so far). A selection:

- **Replace the Supabase SDK with a raw `pg` adapter** — repositories become testable against a local PostgreSQL container; migrated domain by domain behind contract tests
- **Keep the auth adapter outside domain repositories** — domains don't know about the identity provider
- **Test strategy for DDD-lite domains** — in-memory for use cases, Testcontainers for adapters, Supertest for routes
- **Owner resolution and authorization** — a single, explicit model for who can act on what
- **Catalog templates as published snapshots**
- **From VPS to serverless** — first hosted on a VPS hidden behind a Cloudflare Tunnel, then moved to Vercel Functions (stateless requests, signed-cookie sessions, transaction pooler, DB migrations run in CI before each deploy)
- **Product soft delete** *(proposed)* — preserve inventory history when products are removed

## Engineering practices

- Test-driven development, with integration tests on a real database in CI
- CI/CD with GitHub Actions: every push to `main` is tested, then deployed
- 500+ commits, small and incremental
- Runbooks and security audit notes kept alongside the code

## AI-assisted development

xtoque is also my playground for the agentic engineering practices I use at work:

- **Claude Code** as the main coding agent, guided by a curated `CLAUDE.md` (commands, architecture rules, testing conventions)
- **Context engineering**: domain glossary and agent-facing docs (`CONTEXT.md`, `docs/agents/`) so the agent speaks the project's ubiquitous language
- **Custom agent skill with evals** (`scribe`) that audits documentation and ADRs against the actual code to catch stale docs
- **Playwright MCP server** so the agent can open a seeded preview of the app, sign in as each role and visually check UI changes at desktop and phone widths
- **ADRs as specs**: decisions are written down first, then implemented — the agent works from the same written context as a human would

## Contact

Interested in the details? Reach me on [LinkedIn](https://www.linkedin.com/in/arthur-isnard/).
