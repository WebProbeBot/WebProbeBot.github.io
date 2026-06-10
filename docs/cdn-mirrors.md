# GitHub Pages CDN Acceleration Catalog

A catalogue of CDN and proxy services that can serve files from the `WebProbeBot.github.io` repository. Measurements were taken on **2026-06-10** from a single non-China test host; real-world latency varies significantly by client region and ISP.

> **Live benchmark:** open [`speedtest.html`](../speedtest.html) to measure every endpoint listed below from your own network.

---

## 1. URL Template Quick Reference

Using `hello.json` as the example file, the direct GitHub Pages URL is:

```
https://webprobebot.github.io/hello.json
```

To route the same file through any of the providers below, swap the prefix and follow each provider's path template:

| Type | URL template (replace `hello.json` with your path) |
|---|---|
| GitHub Pages (origin) | `https://webprobebot.github.io/hello.json` |
| GitHub raw (origin) | `https://raw.githubusercontent.com/WebProbeBot/WebProbeBot.github.io/main/hello.json` |
| jsDelivr family (`/gh/<user>/<repo>@<ver>/<file>`) | `https://cdn.jsdelivr.net/gh/WebProbeBot/WebProbeBot.github.io@main/hello.json` |
| Statically (`/gh/<user>/<repo>/<branch>/<file>`) | `https://cdn.statically.io/gh/WebProbeBot/WebProbeBot.github.io/main/hello.json` |
| raw.githack (`/<user>/<repo>/<branch>/<file>`) | `https://raw.githack.com/WebProbeBot/WebProbeBot.github.io/main/hello.json` |
| GitHub raw proxy (`/<full-raw-URL>`) | `https://gh-proxy.com/https://raw.githubusercontent.com/WebProbeBot/WebProbeBot.github.io/main/hello.json` |

---

## 2. Global CDNs (Recommended)

### 2.1 GitHub Pages direct (Fastly)
- **URL:** `https://webprobebot.github.io/{file}`
- **Architecture:** Official GitHub Pages on top of the Fastly global CDN.
- **Stability:** ⭐⭐⭐⭐⭐ — this is where the repository is natively hosted.
- **Latency:** 200+ POPs worldwide; the lowest baseline in most regions.
- **When to use:** **The default. You usually don't need to add a CDN on top.**

### 2.2 jsDelivr
- **URL:** `https://cdn.jsdelivr.net/gh/WebProbeBot/WebProbeBot.github.io@main/{file}`
- **Architecture:** Multi-CDN (Cloudflare + Fastly + Bunny + GCore + an in-house Rust edge layer). In early 2026, jsDelivr added a dedicated mainland-China route.
- **Stability:** ⭐⭐⭐⭐ — a 5+ hour expired-certificate incident in May 2024 and two outages of >30 minutes in late 2025 are publicly documented.
- **Caching:** `@main` is cached for 12 hours; `@<commit>` / `@<tag>` is cached permanently.
- **When to use:** Cross-continent distribution, embedding in third-party pages, or any case where you want long-lived immutable URLs.

### 2.3 Statically
- **URL:** `https://cdn.statically.io/gh/WebProbeBot/WebProbeBot.github.io/main/{file}`
- **Architecture:** Multiple CDN providers (Google Cloud / CloudFront / Cloudflare / Fastly / Bunny) but a primary data centre in Germany.
- **Stability:** ⭐⭐⭐ — no major known incidents but global distribution is weaker than jsDelivr.
- **When to use:** A reasonable secondary source when jsDelivr is unavailable.

### 2.4 raw.githack.com / rawcdn.githack.com
- **URLs:**
  - Branch-tracking: `https://raw.githack.com/WebProbeBot/WebProbeBot.github.io/main/{file}`
  - Commit-pinned (permanent cache): `https://rawcdn.githack.com/WebProbeBot/WebProbeBot.github.io/<sha>/{file}`
- **Architecture:** Backed by Cloudflare; the successor to the now-defunct RawGit, running since 2013.
- **Stability:** ⭐⭐⭐⭐ — 13 years of operation with no publicly known major incidents.
- **When to use:** A reliable accelerator for `raw.githubusercontent.com` URLs.

---

## 3. jsDelivr Edge Backends

When `cdn.jsdelivr.net` itself is unreachable, you can force traffic through one specific backend. **The path is identical to the canonical jsDelivr URL — only the host changes.**

| Host | Backend | Use case |
|---|---|---|
| `testingcf.jsdelivr.net` | Cloudflare | Force a CF route |
| `fastly.jsdelivr.net` | Fastly | Force a Fastly route |
| `quantil.jsdelivr.net` | Quantil (Wangsu) | A China-focused edge |
| `jsdelivr.b-cdn.net` | Bunny CDN | Force a Bunny route |

---

## 4. jsDelivr Mirrors in Mainland China

`cdn.jsdelivr.net` lost its mainland-China ICP filing in late 2021 and has been intermittently affected since. Community-run mirrors expose the same path structure under different hostnames; you only need to swap the host.

| Host | Operator | Status (2026-06-10) | Notes |
|---|---|---|---|
| `cdn.jsdmirror.com` | JSDMirror | ✅ Reachable | The most stable of the China-hosted options |
| `jsd.onmicrosoft.cn` | Community | ✅ Reachable | Average latency |
| `jsd.cdn.zzko.cn` | Community | ❌ **Currently unreachable** | A volunteer project — can disappear at any moment |

⚠️ **Community mirrors come and go without warning and should never be relied upon in production.** Use jsDelivr proper with an application-level fallback (see §7), or front the repo with a CDN you own.

---

## 5. GitHub raw Proxies (Not CDNs)

These services proxy `raw.githubusercontent.com`. They are **not** CDNs in the traditional sense — there is no global edge network and no 12-hour cache like jsDelivr, so changes propagate immediately.

| Host | Purpose | Notes |
|---|---|---|
| `gh-proxy.com` | Releases, archives, raw files, `git clone` | Widely used in China; relatively stable |
| `ghproxy.net` | Same as above | A backup option |

URL pattern: `https://gh-proxy.com/https://raw.githubusercontent.com/<user>/<repo>/<branch>/<file>`

**When to use:** `git clone` acceleration, release-asset downloads, and any case where you cannot tolerate jsDelivr's 12-hour cache delay.

---

## 6. Decision Tree

```
Personal or test workload?
├─ Yes → Use https://webprobebot.github.io/<file> directly.
│         It is already on Fastly's global network and is the most reliable
│         free option you can pick. Adding a CDN on top usually adds risk,
│         not speed.
└─ No (you really need acceleration)
    │
    Audience primarily in mainland China?
    ├─ Yes → ① cdn.jsdmirror.com   ② gh-proxy.com
    │        ③ Own ICP filing + a domestic CDN fronting GitHub Pages
    └─ No (global / outside China)
        │
        Do you need an SLA?
        ├─ Yes → Switch hosting: Cloudflare Pages, Vercel, Netlify,
        │         or a paid CDN (CloudFront, Bunny, commercial Fastly).
        └─ No → ① cdn.jsdelivr.net + application-level fallback
                ② Secondary sources: Statically, raw.githack
```

---

## 7. Application-Level Multi-Source Fallback (Recommended Pattern)

**Don't stack CDNs (if jsDelivr is down, your site goes down with it). Instead, define multiple sources and fail over (the site stays up unless *every* source is down):**

```js
const URLS = [
  'https://webprobebot.github.io/hello.json',                                       // primary
  'https://cdn.jsdelivr.net/gh/WebProbeBot/WebProbeBot.github.io@main/hello.json',  // backup 1
  'https://raw.githack.com/WebProbeBot/WebProbeBot.github.io/main/hello.json',      // backup 2
];

async function fetchWithFallback() {
  for (const url of URLS) {
    try {
      const r = await fetch(url, { signal: AbortSignal.timeout(3000) });
      if (r.ok) return r.json();
    } catch {}
  }
  throw new Error('all sources down');
}
```

---

## 8. References

- jsDelivr status page: <https://status.jsdelivr.com/>
- raw.githack FAQ: <https://raw.githack.com/faq>
- gh-proxy: <https://gh-proxy.com/>
- JSDMirror: <https://github.com/jsdmirror/JSDMirror>
