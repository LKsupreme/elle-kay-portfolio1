# ELLE KAY — International AI + 3D Creative Portfolio

Cloudflare-first React/Vite portfolio with Cloudflare Workers, D1, Turnstile and Resend-ready APIs. This version does not require Cloudflare R2.

## 1. Install

```bash
npm install
```

## 2. Local development

```bash
npm run dev
```

The public site runs on Vite. API routes are implemented in `functions/worker.ts` for Cloudflare deployment.

## 3. Environment / secrets

Copy `.env.example` as needed. Never commit secrets.

For production use Wrangler secrets:

```bash
wrangler secret put SESSION_SECRET
wrangler secret put TURNSTILE_SECRET_KEY
wrangler secret put RESEND_API_KEY
wrangler secret put CONTACT_EMAIL
```

`VITE_TURNSTILE_SITE_KEY` and other public values may be exposed to the browser. The Turnstile secret, Resend key and session secret must remain server-side.

## 4. D1

Create the database and put its ID in `wrangler.toml`:

```bash
wrangler d1 create elle-kay-db
```

Then run:

```bash
npm run db:migrate
npm run db:migrate:remote
```

## 5. Media storage without R2

This free-plan-friendly build does **not** require a Cloudflare R2 subscription. Media records store URLs in D1.

For local/static portfolio media, put files under `public/media/` and reference them from the CMS as `/media/filename.webp`. Because Cloudflare Workers Static Assets serves the `dist` directory, these files deploy with the site.

For large videos, you can use an external video/CDN URL and register that URL in the Media Library. The media abstraction keeps the CMS independent of the storage provider.

The Media Library therefore registers/reuses media URLs rather than attempting to upload binary files into R2.

## 6. Turnstile

Create a Cloudflare Turnstile site for `ellekaybuilds.com`, put the public site key in the public configuration and the secret in Wrangler.

The server validates the Turnstile token. Browser-only validation is not trusted.

## 7. Resend

Set `RESEND_API_KEY` and `CONTACT_EMAIL`. The contact endpoint stores every submission in D1 and attempts email delivery when configured.

## 8. Admin setup

The schema includes `admin_users` and `admin_sessions`. Before production, create an admin user whose password hash is an HMAC-SHA256 digest generated with the same `SESSION_SECRET` and the desired password, then insert it into D1. Do not put plaintext passwords in source control.

Example using Web Crypto-compatible tooling or a one-off Worker script is recommended.

## 9. Database migration

Migrations live in `migrations/` and seed the portfolio content specified in the build brief. Projects explicitly marked as drafts remain unpublished.

## 10. Production build

```bash
npm run build
```

## 11. Cloudflare deployment

Update `wrangler.toml` with the actual D1 database ID and then:

```bash
npm run deploy
```

## 12. Custom domain

Attach `ellekaybuilds.com` to the deployed Worker in Cloudflare. Keep the intended production URL in `SITE_URL`.

## 13. CMS

Open `/admin`. The CMS supports authenticated project listing, creation and editing, including title, slug, role, description, publication and featured status. The schema/API also supports the broader media, page, service, experience, SEO and settings model required for expanding the dashboard.

## Architecture

- `src/` — public React application
- `src/admin/` — CMS UI
- `functions/worker.ts` — Cloudflare Worker API and SPA fallback
- `migrations/` — D1 schema + seed data
- `public/` — static assets

## Important content rules

Do not invent clients, awards, certifications, project sizes, budgets, licenses, outcomes, team sizes or unsupported locations. The supplied master build brief is the source of truth for portfolio facts.

## Current implementation notes

The repository includes the public React application, D1 schema/seed content, Worker API, session-based admin authentication, project CRUD, URL-based media library without R2, contact persistence/email foundation, SEO-ready data fields, and Cloudflare deployment configuration. The design intentionally keeps project content and media references data-driven.

For production, complete the environment-specific Cloudflare setup (D1 database ID,  Turnstile credentials, Resend credentials and an admin user) before publishing.
