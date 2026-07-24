# Delta Structures — deployable static site

This is the site **unbundled** from the previous single-file artifact-bundler build.
Push the contents of this folder to the repo root and redeploy.

## Repo layout (all paths are root-relative)

```
index.html          the real site (2.8 MB, was 8.4 MB)
_headers            Cloudflare Pages response headers (CSP, caching, security)
robots.txt
sitemap.xml
assets/             110 images + hero video + og-image.jpg
js/                 gsap.min.js, lenis.min.js
```

## Cloudflare Pages build settings (unchanged)

| Setting | Value |
|---|---|
| Framework preset | None |
| Build command | *(empty)* |
| Build output directory | `/` |

Static site — no build step. `_headers` is read automatically by Pages.

## What changed and why

1. **Removed the artifact-bundler wrapper.** The old `index.html` was a loader that
   held the entire site as a JSON string in `<script type="__bundler/template">`,
   unpacked it at runtime and swapped `document.documentElement`. It also installed a
   global error sink that painted the red `[bundle] …` banner **to real visitors** for
   any page-level JS error — including errors the site did not cause. That banner is
   what was on screen. It cannot occur any more; the sink no longer exists.

2. **Fixed the Content-Security-Policy.** The old `<meta>` CSP had
   `script-src 'self' 'unsafe-inline' 'unsafe-eval' blob:` with **no `data:`**, while
   GSAP, Lenis and a leftover scaffold were all loaded from `data:` URLs — so all three
   were refused on the live site and animations/smooth-scroll were dead. Those libraries
   are now real files under `/js/`, so `script-src 'self'` covers them. CSP now lives in
   `_headers` instead of a `<meta>` tag.

3. **Dropped a 55 KB `<image-slot>` authoring scaffold** that shipped by mistake. It
   registers an editor custom element and talks to a "host bridge" that does not exist
   in production. Nothing on the site uses it.

4. **Extracted 110 inlined base64 images and the 1.1 MB hero video to real files.**
   They are now cached separately (1-year immutable) instead of being re-downloaded
   inside the HTML on every visit.

5. **Fixed `og:image` / `twitter:image`.** They pointed at a `data:image/webp` URI.
   Social crawlers cannot fetch data URIs, so link previews were blank. They now point
   at `https://www.deltastructures.in/assets/og-image.jpg` (absolute, JPEG, 1200 px).

## Post-deploy check

Open the live URL with DevTools console open. Expected: **no red banner, no CSP
refusals, no JS errors.** `window.gsap` and `window.Lenis` should both be defined.
The map and Google Analytics stay inert until someone clicks **Accept** on the cookie
banner — that is the DPDP behaviour working as intended.
