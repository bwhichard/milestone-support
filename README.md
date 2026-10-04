# Milestone website

Static site for the iOS app [Milestone](https://apps.apple.com/us/app/milestone-birthdays/id6759117799), served at https://trymilestone.app. No build step, no dependencies; the repo root is the site.

- `index.html` – landing/support page
- `privacy.html` – privacy policy (served at `/privacy`)
- `s/index.html` – landing page for share links
- `.well-known/apple-app-site-association` – Apple Universal Links file
- `_headers` – Cloudflare response headers (honored by Workers static assets)
- `.assetsignore` – keeps `.git`, `README.md`, env files and tooling caches out of the published site
- `.env.schema` – varlock schema for the Cloudflare API token (no secrets)

## Share links

The app shares birthdays/anniversaries as:

    https://trymilestone.app/s/#v1.<base64url(raw-deflate(JSON))>

The data lives in the URL fragment, which browsers never send, so the server never sees it. `s/index.html` decodes it client-side (only to show a summary and up to 8 names, via `textContent`). Do not add analytics, logging, or third-party scripts to this site.

## Universal Links (AASA)

iOS opens the app for `/s/*` only if `/.well-known/apple-app-site-association` is valid: app ID `HWLGH75UFB.com.brandonwhichard.Milestone`, component `/s/*`. The file has no extension and must be served as `application/json` with no redirects, which `_headers` guarantees on Cloudflare (GitHub Pages ignores `_headers`). `_headers` also marks `/s/*` as noindex, no-referrer, no-store.

## Hosting

Cloudflare Worker with static assets (project `milestone-support`, account "Personal Sites"), connected to this repo via Git. The dashboard build runs `npx wrangler deploy` with an auto-generated config (assets directory `.`); there is no build step and no config file in the repo. Push to `main` to deploy. No `CNAME` file is needed (that is only for GitHub Pages).

- Custom domains on the Worker: `trymilestone.app` and `www.trymilestone.app`. The workers.dev URLs are turned off.
- `www` redirects to the apex: a zone Redirect Rule ("www to apex", ruleset `Redirects`, 301, path and query preserved). The apex is never redirected, because the AASA must be served with no redirects.
- Email: Cloudflare Email Routing is enabled. A catch-all forwards every address at the domain to `bwhichard@gmail.com`.
- `.app` is on the browser HSTS preload list, so Always Use HTTPS is intentionally left off.
- GitHub Pages (`bwhichard.github.io/milestone-support/`) is still on, so URLs in already-released builds keep working. It can be disabled later.

## Managing Cloudflare with `cf`

The `cf` CLI authenticates with a scoped API token stored in 1Password (`SDT_DEV` / `MILESTONE_CLOUDFLARE_TOKEN`, field `credential`). `.env.schema` declares `CLOUDFLARE_API_TOKEN`; the git-ignored `.env.local` resolves it through the 1Password service account token in `~/.env.1p-token`. Run commands through varlock:

    varlock run -- cf rulesets account-rulesets list -z trymilestone.app

The token is limited to the `trymilestone.app` zone and the Personal Sites account (Workers, Single Redirect, DNS, Zone Settings, Email Routing). Rotate it when it expires.

## Verify

    curl -sI https://trymilestone.app/.well-known/apple-app-site-association   # 200, application/json, no 3xx
    curl -s https://app-site-association.cdn-apple.com/a/v1/trymilestone.app   # Apple CDN copy, may lag ~24h
    curl -sI https://www.trymilestone.app/privacy                              # 301 to the apex
