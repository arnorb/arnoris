# AGENTS.md

Personal site for Arnór Bogason at https://arnor.is: Astro, built as a static site and served from Cloudflare Workers static assets. See `README.md` for the stack, layout and DNS.

Agent config (tracker, domain docs, notes and plans folders, commands): see `docs/agents/`.

## Ground rules

- **Keep it minimal.** The site is deliberately a placeholder. Don't add a CMS, i18n, a UI framework, client JavaScript or new dependencies unless asked. Payload CMS is the likely choice later, and that decision is still open.
- **English only** on the site.
- **Pushing to `main` deploys to production** through Workers Builds. Push only when asked.
- **Use pnpm, never npm or yarn.** After changing `package.json`, run `pnpm install` and commit `pnpm-lock.yaml`. CI installs with `--frozen-lockfile`, and a stale lockfile fails the build.
- **Before handing back,** run `pnpm astro check` and `pnpm build`. Both must pass.
- **TypeScript is pinned to 6.** `astro check` doesn't support TypeScript 7 yet.
- **The `@emnapi/runtime` override in `pnpm-workspace.yaml` is intentional.** Without it, `astro check` crashes. Remove it only once `astro check` passes without it.

## Design

- **Colors** are CSS custom properties on `:root` in `src/layouts/Base.astro`, redefined under `prefers-color-scheme: dark`. Add colors as tokens there, never as hard-coded values in pages.
- **Font:** one font file, `public/fonts/inter-var-latin.woff2`, with a metric-matched Arial fallback. Any typeface change must keep the Icelandic letters (á ð þ æ ö and the rest).
- **Accessibility:** keep `lang="en"`, one `h1` per page, visible `:focus-visible` styles, and WCAG AA contrast in both themes.

## Don't touch without asking

- Cloudflare DNS, redirect rules or the Worker's custom domain.
- The mail records (MX, SPF, DKIM, DMARC) and `_atproto`.
- The Fathom site ID.
