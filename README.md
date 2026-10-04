# Milestone website

Static site for the iOS app [Milestone](https://apps.apple.com/us/app/milestone-birthdays/id6759117799), served at https://trymilestone.app. No build step, no dependencies; the repo root is the site.

- `index.html` – landing/support page
- `privacy.html` – privacy policy (served at `/privacy`)
- `s/index.html` – landing page for share links
- `.well-known/apple-app-site-association` – Apple Universal Links file
- `_headers` – Cloudflare Pages response headers

## Share links

The app shares birthdays/anniversaries as:

    https://trymilestone.app/s/#v1.<base64url(raw-deflate(JSON))>

The data lives in the URL fragment, which browsers never send, so the server never sees it. `s/index.html` decodes it client-side (only to show a summary and up to 8 names, via `textContent`). Do not add analytics, logging, or third-party scripts to this site.

## Universal Links (AASA)

iOS opens the app for `/s/*` only if `/.well-known/apple-app-site-association` is valid: app ID `HWLGH75UFB.com.brandonwhichard.Milestone`, component `/s/*`. The file has no extension and must be served as `application/json` with no redirects, which `_headers` guarantees on Cloudflare Pages (GitHub Pages ignores `_headers`). `_headers` also marks `/s/*` as noindex, no-referrer, no-store.

## Deploy

Cloudflare, connected to this repo via Git. The dashboard builds it as a Worker with static assets (`npx wrangler deploy`, auto-generated config, assets directory `.`); there is no build step. `.assetsignore` keeps `.git`, `README.md` and build files out of the published assets. Custom domain `trymilestone.app` is attached under the Worker's Settings → Domains & Routes. No `CNAME` file is needed (that is only for GitHub Pages). Push to `main` to deploy.

Verify after deploy:

    curl -sI https://trymilestone.app/.well-known/apple-app-site-association   # 200, application/json, no 3xx
    curl -s https://app-site-association.cdn-apple.com/a/v1/trymilestone.app   # Apple CDN copy, may lag ~24h
