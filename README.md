# WebProbeBot.github.io

Source for the **WebProbeBot GitHub Pages site** — a client-side benchmark for fetching static files from a GitHub repository through multiple CDN endpoints.

Live: <https://webprobebot.github.io>

## Features

- **CDN Speed Test** ([`speedtest.html`](speedtest.html)) — one-click benchmark across 14 endpoints (GitHub Pages, raw.githubusercontent, jsDelivr, Statically, raw.githack, four jsDelivr edge backends, three China-hosted jsDelivr mirrors, and two GitHub raw proxies).
- **CDN Mirror Catalog** ([`docs/cdn-mirrors.md`](docs/cdn-mirrors.md)) — URL templates, stability notes, decision tree and the recommended multi-source fallback pattern.
- **Connectivity fixture** ([`hello.json`](hello.json)) — a tiny JSON file used both as the default benchmark target and as a public reachability test.

## Local preview

Open `index.html` directly in a browser, or serve the directory:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploy

Pushing to `main` publishes via GitHub Pages automatically — no build step required.

## License

The code in this repository is published as-is for reference. Third-party CDN endpoints listed in the catalog belong to their respective operators; uptime and availability are not guaranteed.
