# stacktician.com — engineering conventions

Static site for Stacktician Limited. One `index.html`, no build step, no framework. Every push
to `main` deploys through Cloudflare Pages (project `stacktician-com`). `_headers` and
`_redirects` are live configuration, not documentation.

## Non-negotiables (each one came from a real regression on 15–16 Oct 2026)

- **No external requests.** Fonts are self-hosted in `fonts/`; icons are inline SVG or data URIs.
  Adding any third-party resource (analytics, fonts, scripts, images) means: allow it in the
  CSP, load it with `crossorigin`, and confirm it is served with CORS/CORP headers, or
  `Cross-Origin-Embedder-Policy: require-corp` blocks it silently.
- **CSP is hash-pinned.** `script-src` lists the SHA-256 of each inline `<script>` in
  `index.html`. After any change to an inline script, run from `C:/dev/ai/domains`:
  `uv run python -B csp_script_hashes.py C:/dev/ai/stacktician.com/index.html`
  and paste the new hashes into `_headers`. Never add `'unsafe-inline'` to `script-src`, not
  even as the "older browsers" fallback Lighthouse suggests.
- **Trusted Types are enforced** (`require-trusted-types-for 'script'`). Scripts must not use
  `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, `eval` or `new Function`.
  Use `textContent`, `classList`, `dataset` and DOM constructors.
- **Styling never depends on JavaScript.** Motion and reveal rules are gated on `html.js`, so
  a blocked or failed script still shows every element. Keep it that way: no `opacity:0`
  without an `html.js` prefix.
- **Read layout before writing styles.** Take `scrollY`, `getBoundingClientRect` and similar
  readings first, then change classes or attributes. The reverse order forces a reflow that
  Lighthouse reports.
- **Headers Pages adds by default are removed on purpose**: `! Access-Control-Allow-Origin`
  under both `/*` and `/fonts/*`. Keep those lines.
- **Fonts are cached immutable for a year.** If a font file changes, rename it and update
  the `@font-face` and preload references; never overwrite in place.
- **Redirect split:** path-level links (`/github`) go in `_redirects`; domain-level redirects
  (www → apex) are Cloudflare zone Redirect Rules, because the file cannot match hostnames.
- **Legal footer stays:** registered name, company number 17518638, place of registration and
  registered office are required on the site by the Companies Act.

## Cloudflare zone settings that must not drift

- Email Address Obfuscation: **off** (it rewrites mailto links and injects a script the CSP
  blocks, leaving visitors a placeholder instead of hello@stacktician.com).
- SSL/TLS mode: Full (strict). Bot Preference Sync: off, so `robots.txt` is served as committed.
- DNS: proxied CNAME `@` and `www` → `stacktician-com.pages.dev`; MX fwd1/fwd2.porkbun.com;
  SPF and `_dmarc` TXT. Registrar and email forwarding stay at Porkbun.

## Before calling a change done

1. Local CSP check: copy `index.html` to a throwaway file with the `_headers` CSP inserted as a
   `<meta http-equiv="Content-Security-Policy">` (drop `frame-ancestors`), load it over the
   preview server, confirm `document.body.classList.contains("is-ready")`, the footer year is
   set, and the console is empty. Delete the throwaway file.
2. Phone width (375 px): no horizontal overflow, both fonts loaded, nothing left at opacity 0.
3. After deploy: https://securityheaders.com must stay **A+**; https://pagespeed.web.dev
   mobile must stay 100 on all four categories; browser console on the live page empty.
4. `sitemap.xml` `lastmod` updated on content changes.

## Layout

- `index.html`, `404.html`, `robots.txt`, `sitemap.xml`, `.well-known/security.txt`
- `_headers` (security headers, font caching), `_redirects` (short links only)
- `fonts/` self-hosted woff2; `brand/` the mark as SVG source and 512 px PNG
- `TODO.md` is gitignored: Arun's private working list, never committed
