# Cutover: arnor.is → Cloudflare Worker `arnoris`

Arnór runs these steps. The new site is live at the preview **https://arnoris.arnor.workers.dev** (version `152e6ed8-1daa-4e4c-ae6c-2c583a4fb651`, deployed 2026-11-08).

## DNS before the switch (recorded 2026-11-08)

Public lookups only. Both hostnames are proxied by Cloudflare, so the Netlify targets behind them don't show up in DNS lookups.

| Name | Type | Public answer |
| --- | --- | --- |
| `arnor.is` | A | 172.67.169.100, 104.21.87.110 (Cloudflare proxy) |
| `arnor.is` | AAAA | 2606:4700:3031::6815:576e, 2606:4700:3033::ac43:a964 (Cloudflare proxy) |
| `www.arnor.is` | A / AAAA | same Cloudflare proxy IPs |
| `arnor.is` | MX | `10 mail.protonmail.ch.` |
| `arnor.is` | TXT | `v=spf1 include:_spf.protonmail.ch mx ~all` |
| `arnor.is` | TXT | `protonmail-verification=913f80a9cd9bc0debb09c767efd976a972139121` |
| NS | | `julio.ns.cloudflare.com`, `lady.ns.cloudflare.com` |

- [x] **0. Back up the zone.** In the Cloudflare dashboard, go to arnor.is, then DNS, then Records, and use Import and Export to export the zone file. Keep it to roll back with; it contains the real Netlify targets.

## Steps

- [c] **1. Rxmove the apex record that points to Netlify.** Under DNS, delete the `arnor.is` record (A or CNAME to Netlify). Cloudflare won't attach a Worker custom domain while another record exists for that hostname. The site is down for a few seconds between this step and step 2.
- [x] **2. Attach the custom domain.** Go to Workers & Pages, then `arnoris`, then Settings, then Domains & Routes, then Add, then Custom domain, and enter `arnor.is`. Cloudflare creates the DNS record and certificate.
- [x] **3. Repoint `www`.** Edit the `www` record so it no longer points to Netlify: type `AAAA`, content `100::`, proxied (orange cloud). This is a placeholder address so the redirect rule in step 4 can fire.
- [x] **4. Redirect `www` to the apex.** Go to Rules, then Redirect Rules, then Create (a Single Redirect):
  - When incoming requests match: custom filter expression `(http.host eq "www.arnor.is")`
  - Then: Dynamic, expression `concat("https://arnor.is", http.request.uri.path)`, status `301`, with **Preserve query string** on.
- [x] **5. Confirm mail is untouched.** The MX and both TXT records in the table above should still be there, unchanged.
- [x] **6. Smoke-check** (from a terminal):
  ```sh
  curl -sI https://arnor.is/ | head -1                                # 200
  curl -sI https://www.arnor.is/some/path?x=1 | grep -i location      # https://arnor.is/some/path?x=1
  curl -sI https://arnor.is/verkefni/bjarg | grep -i location         # /  (301)
  curl -sI https://arnor.is/does-not-exist | head -1                  # 404
  ```
- [x] **7. Check Fathom.** Open https://arnor.is in a browser, and a pageview should appear in Fathom (site `KJMPLXTI`) within a minute.
- [ ] **8. Delete the Netlify site** in the Netlify dashboard, and remove the custom domain from it first if it's still attached.

## Rollback

Delete the `arnoris` custom domain in Workers and the redirect rule, then restore the apex and `www` records from the zone export made in step 0.

## Verification done before handover (2026-11-08)

| Check | Result |
| --- | --- |
| `pnpm astro check` | 0 errors, 0 warnings, 0 hints |
| `pnpm build` | 2 pages: `index.html`, `404.html` |
| `dist/index.html` | `lang="en"`, one `h1`, one-liner, `mailto:arnor@arnor.is`, `https://github.com/arnorb`, `https://www.linkedin.com/in/arnorb`, title, description, Open Graph tags, canonical `https://arnor.is/`, Fathom `data-site="KJMPLXTI"` |
| `dist/404.html` | links to `/`, `noindex` |
| Browser, 1440px light and dark | renders; Inter 100–900 loaded; á ð þ æ ö ý in Inter |
| Browser, 320px | no horizontal scroll, on both the home page and the 404 page |
| Fathom script | `window.fathom` present in the browser |
| workers.dev: `/` | 200 |
| workers.dev: `/does-not-exist` | 404 with the not-found page |
| workers.dev: `/verkefni`, `/verkefni/bjarg`, `/um-mig/`, `/hafa-samband` | 301 to `/` |

Old-site routes found by crawling `www.arnor.is` (it has no sitemap): `/verkefni`, `/verkefni/bjarg`, `/verkefni/hsveitur`, `/verkefni/unak`, `/um-mig`, `/hafa-samband`. The `_redirects` file covers them all, with and without a trailing slash.

Dark mode was checked by applying the dark tokens directly. The browser window's color scheme couldn't be switched from automation, so check it once on a device that is set to dark mode.
