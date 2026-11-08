# Spec: arnor.is name card

## Problem statement

People who look up Arnór Bogason at `arnor.is`, such as recruiters, past clients, peers and friends, land on an outdated Nuxt 2 site on Netlify. It describes him as "vefhönnuður", still has the default "My brilliant Nuxt.js project" meta description, and loads a Netlify Identity widget it never uses. It doesn't say what he does now or how to reach him. Arnór now works full-time as a Design Engineer at Húsvör, so he needs a page that is correct today and stays correct without upkeep, on a codebase that can later grow into writing, a portfolio and a playground.

## Solution

A new site, built from scratch, that is a single English name card at `arnor.is`. It shows his name, one line saying who he is, and three ways to reach him: email, GitHub and LinkedIn. The look is minimal and typographic: one self-hosted variable typeface, generous whitespace, and light and dark themes that follow the system setting. It's an Astro static site served from Cloudflare Workers static assets, with Fathom analytics. `www.arnor.is` and every path from the old site redirect to the new page, and the Netlify site is retired.

## User stories

1. As a visitor, I want to see Arnór's name as the page's main heading, so that I know I've reached the right person.
2. As a visitor, I want to read "Design Engineer at Húsvör, based in Reykjavík.", so that I know what he does now and where he is.
3. As a visitor, I want a link that opens my mail app addressed to `arnor@arnor.is`, so that I can contact him straight away.
4. As a visitor, I want a link to `https://github.com/arnorb`, so that I can see his code.
5. As a visitor, I want a link to `https://www.linkedin.com/in/arnorb`, so that I can see his work history.
6. As a visitor on a phone, I want the card to read comfortably at 320px wide with no horizontal scrolling, so that I can use it anywhere.
7. As a visitor whose system uses dark mode, I want the page in a dark theme, so that it matches my setting.
8. As a keyboard user, I want to tab through the three links with a visible focus ring, so that I can reach every contact method without a mouse.
9. As a screen-reader user, I want the page to declare `lang="en"`, have one `h1` and give each link a clear name (e.g. "Email", "GitHub", "LinkedIn"), so that the page reads correctly.
10. As a visitor on a slow connection, I want the text to show at once in a fallback font while the web font loads, so that I'm never looking at a blank page.
11. As someone sharing the link in Slack, iMessage or LinkedIn, I want the preview to show "Arnór Bogason" and the one-liner, so that the link explains itself.
12. As a visitor who searches for his name, I want the result to show the title "Arnór Bogason" and the one-liner as the description, so that I know it's his site.
13. As a visitor, I want a recognisable icon in my browser tab, so that I can find the tab again.
14. As a visitor who types `www.arnor.is`, I want to be sent to `https://arnor.is/` with the path kept, so that both addresses work.
15. As a visitor following an old link to a page on the previous site, I want to land on the new home page instead of an error, so that old links still lead somewhere useful.
16. As a visitor who mistypes a path, I want a short "Page not found" page in the same style with a link home, so that I can recover.
17. As Arnór, I want visits counted in my existing Fathom account (site ID `KJMPLXTI`) without cookies or a consent banner, so that I can tell whether anyone visits.
18. As Arnór, I want the email and Proton mail records on `arnor.is` to keep working through the switch, so that I don't lose mail.

## Implementation decisions

**Stack**
- Astro, output `static`. Package manager pnpm. Commands are in `docs/agents/commands.md`.
- No UI framework, CMS, i18n or client-side JavaScript other than the Fathom script.

**Pages**
- **Home (`/`)**, one page:
  - an `h1` with "Arnór Bogason"
  - a paragraph with "Design Engineer at Húsvör, based in Reykjavík."
  - a list of three links: Email (`mailto:arnor@arnor.is`), GitHub, LinkedIn.
  - External links open in the same tab.
- **Not found (`404`)**: "Page not found" plus a link to `/`, on the same layout.
- **Shared layout component**, which holds:
  - `<html lang="en">`, the color tokens, and the `@font-face` declaration
  - a `<title>` and a `<meta name="description">` set per page
  - Open Graph tags: `og:title`, `og:description`, `og:url` = `https://arnor.is/`, `og:type` = `website`
  - `twitter:card` = `summary`. There is no social image.
  - a canonical link to the `https://arnor.is` URL
  - the favicon
  - the Fathom script.
- Home page title: "Arnór Bogason". Home description: "Design Engineer at Húsvör, based in Reykjavík."

**Design**
- **Typeface:** one variable typeface, self-hosted as a single WOFF2 file with `font-display: swap`. It must have a SIL Open Font License, a weight axis, and the Icelandic letters á é í ó ú ý ð þ æ ö in upper and lower case. Inter is the default. The builder may propose another font that meets the same criteria, and Arnór approves the choice on the preview.
- **Fallback:** a system font stack, adjusted with `size-adjust` so the text doesn't jump when the web font arrives.
- **Colors** are tokens on `:root`: background, text, muted text, link and focus ring. They're redefined under `@media (prefers-color-scheme: dark)`. There is no theme toggle. Text and link colors meet WCAG AA contrast in both themes.
- **Layout:** one column, a max width of about 36rem, left-aligned, and at least a 16px gutter at phone width.
- **No imagery or motion.** Link hover and `:focus-visible` states are visible in both themes.
- **Favicon:** an SVG monogram "A" built from the color tokens, with dark-mode support inside the SVG.

**Analytics**
- Fathom embed: `https://cdn.usefathom.com/script.js` with `data-site="KJMPLXTI"`, loaded `defer`. It sends no custom events.

**Hosting**
- Cloudflare Workers static assets: a Wrangler config that serves the Astro build output (`dist`) as assets, with no Worker script. Not-found handling uses the `404-page` mode, so the built 404 page is served with status 404.
- Deploys run as `wrangler deploy` from Arnór's machine. A `deploy` script in `package.json` runs the build and then the deploy.
- The first deploy goes to the `workers.dev` preview URL. The custom domain is attached only during cutover.

**Redirects**
- **Old-site paths:** before cutover, list every route of the current `www.arnor.is` site, using its sitemap if there is one and a crawl of its internal links if not. Each route except `/` gets a `301` to `/` in the `_redirects` file served by Workers static assets. This covers the agreed "redirect old URLs that get traffic" by redirecting all of them, because a one-page site has nowhere else to send them.
- **`www.arnor.is`:** a Cloudflare Single Redirect rule on the `arnor.is` zone: `301` to `https://arnor.is` with the path and query string kept. The `www` DNS record stays proxied so the rule fires.

**Cutover**
- Live DNS changes and deleting the Netlify site are Arnór's to run. The builder writes the steps as a checklist (`.scratch/name-card/cutover.md`):
  1. Attach `arnor.is` as a custom domain to the Worker.
  2. Point `www` at the zone (proxied) and create the Single Redirect rule.
  3. Remove the old DNS records that point to Netlify.
  4. Confirm the MX, SPF and Proton verification TXT records are untouched.
  5. Smoke-check `https://arnor.is`, `https://www.arnor.is/some/path` and one old route.
  6. Delete the Netlify site.
- Before step 1, the builder records the current DNS records in the checklist so the change can be rolled back.

**Repo**
- The code lives in the existing public repo `arnorb/arnoris`, which was empty as of 2026-11-08. It's the `origin` remote, and `main` is the default branch.
- `.gitignore` covers `node_modules`, `dist`, `.wrangler`, `.astro` and `.omc`.

## Testing decisions

- No automated tests, as Arnór decided for a single static page. No test runner is installed.
- Before handing over, the builder checks the following and records the results in the cutover checklist:
  - `pnpm build` and `pnpm astro check` pass with no errors.
  - **Built `dist/index.html`:**
    - has `lang="en"`, one `h1`, the one-liner, and the three link `href`s exactly as listed in the user stories
    - has the title, description, Open Graph and canonical tags
    - has the Fathom script with `data-site="KJMPLXTI"`.
  - **Built `dist/404.html`** exists and links to `/`.
  - **On the `workers.dev` preview:**
    - the page renders at 320px and 1440px wide, in light and dark mode
    - Icelandic characters render in the web font
    - `/does-not-exist` returns status 404 with the not-found page
    - one route from `_redirects` returns `301` to `/`.
  - **On the Fathom dashboard,** a pageview shows up from the preview, or from the live site after cutover if Fathom restricts site domains.

## Out of scope

- Writing, a portfolio, a playground, or any content collections.
- Payload or any other CMS. That integration decision is deferred until a CMS is actually adopted.
- Icelandic or bilingual content.
- A visual identity beyond the typeface, color tokens and monogram favicon.
- A social preview image.
- Automated tests and CI.
- Deploying from GitHub (Workers Builds or Actions).
- Changes to the Proton mail setup.

## Open questions

None.

## Further notes

- The current site (as of 2026-11-08): Nuxt 2 on Netlify. `arnor.is` returns a `301` to `www.arnor.is`. DNS is on Cloudflare (`julio`/`lady.ns.cloudflare.com`), and mail goes through Proton (`mail.protonmail.ch`, SPF `include:_spf.protonmail.ch`).
- Earlier versions of the site live in the public repos `arnoris-old`, `arnoris-v8` and `arnor-is-2020`. Leave them as they are.
- Decisions come from the grill session on 2026-11-08 that Arnór confirmed.
