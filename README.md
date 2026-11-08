# arnor.is

Personal site of Arnór Bogason, live at [arnor.is](https://arnor.is). For now it is a one-page name card; it will grow into writing, portfolio and playground later.

## Stack

- [Astro](https://astro.build), built as a static site. No UI framework, no CMS, and no client JavaScript apart from analytics.
- Hosted on [Cloudflare Workers static assets](https://developers.cloudflare.com/workers/static-assets/) as the Worker `arnoris`.
- [Fathom](https://usefathom.com) analytics (site ID `KJMPLXTI`). Cookieless, so there is no consent banner.
- Inter variable font, self-hosted (`public/fonts/`, SIL Open Font License).

## Develop

Requires Node 22+ and pnpm (the version is pinned in `package.json` under `packageManager`).

```sh
pnpm install
pnpm dev          # dev server on http://localhost:4321
pnpm build        # static build to dist/
pnpm astro check  # typecheck
```

## Deploy

A push to `main` builds and deploys through Cloudflare Workers Builds. Pushes to other branches get preview URLs.

To deploy by hand from a machine that's logged in to Wrangler:

```sh
pnpm run deploy
```

## Layout

| Path | What |
| --- | --- |
| `src/layouts/Base.astro` | The HTML shell: meta tags, color tokens (light and dark), font, Fathom script |
| `src/pages/index.astro` | The name card |
| `src/pages/404.astro` | The not-found page, served with status 404 |
| `public/_redirects` | 301s from the old site's routes to `/` |
| `wrangler.jsonc` | Worker config: serves `dist/`, with 404-page handling |
| `docs/agents/` | Config for coding agents (see `AGENTS.md`) |

## Domains and DNS

DNS for `arnor.is` is on Cloudflare.

- **`arnor.is`** is a custom domain on the `arnoris` Worker.
- **`www.arnor.is`** 301-redirects to `https://arnor.is`, keeping the path and query string. A Cloudflare Single Redirect rule does this, not the Worker.
- **Mail** goes through Proton. Leave the MX, SPF, DKIM (`protonmail*._domainkey`, DNS only, never proxied) and DMARC (`_dmarc`) records alone.
- **`_atproto`** holds the Bluesky handle `@arnor.is`. Keep it.
