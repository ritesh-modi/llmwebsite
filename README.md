# thefirstbookonllm.com — static website

Single-page landing site for *Inside Large Language Models*, Volumes I and II. Pure HTML and CSS. No build step. Deploys to Azure Static Web Apps in a few minutes.

## Files

```
website/
├── index.html                              main page (with JSON-LD structured data)
├── styles.css                              styles (book's four-color palette)
├── staticwebapp.config.json                Azure SWA config (routing, headers, caching, CSP)
├── robots.txt                              search + AI crawler directives, sitemap pointer
├── sitemap.xml                             single URL with image siblings for Vol I, Vol II, author
├── site.webmanifest                        PWA manifest for "add to home screen" + theme color
├── Inside-LLMs-Vol1-Cover-2400x3000.jpg    Volume I cover (LCP, preloaded)
├── Inside-LLMs-Vol2-Cover-2400x3000.jpg    Volume II cover
├── ritesh.jpeg                             author headshot (480x480)
├── diagrams/                               seven SVGs lifted from the book chapters
│   ├── bpe.svg              ch3  Byte-Pair Encoding
│   ├── attention.svg        ch5  Single-head attention
│   ├── gpt.svg              ch7  Complete GPT architecture
│   ├── autoregressive.svg   ch9  One-token-at-a-time generation
│   ├── kvcache.svg          ch10 KV cache growth
│   ├── lora.svg             ch12 LoRA architecture
│   └── qlora.svg            ch12 QLoRA memory budget
└── README.md                               this file
```

## Local preview

```bash
cd website
python3 -m http.server 8080
open http://localhost:8080
```

That's it. No dependencies, no bundler, no toolchain.

## Deploy to Azure Static Web Apps

You need an Azure subscription, a GitHub account, and this `website/` folder pushed to a GitHub repo. Static Web Apps wires up CI/CD automatically on the first deploy.

### Option A: deploy from GitHub (recommended)

1. Push the `website/` folder to a GitHub repo. Easiest is a dedicated repo (`thefirstbookonllm-com`) so `website/` becomes the repo root, but it can also live inside the existing `llm/` repo as a subfolder.
2. In the Azure portal, create a new resource: **Static Web App**.
3. Connect the GitHub repo and branch.
4. In the **Build Details** step, set:
   - **Build Presets**: *Custom*
   - **App location**: `/` if `website/` is the repo root, otherwise `/website`
   - **Output location**: leave empty (no build step)
5. Click **Review + Create**. Azure pushes a `.github/workflows/azure-static-web-apps-*.yml` file to the repo and runs the first deploy.
6. Once the workflow shows green, the site is live at the auto-assigned `*.azurestaticapps.net` URL.

### Option B: deploy with the SWA CLI (no Git needed)

```bash
npm install -g @azure/static-web-apps-cli
swa login
swa deploy ./website --env production
```

Useful for quick prototyping or when the site is not in a Git repo yet.

## Custom domain — thefirstbookonllm.com

After the site is deployed and reachable on `*.azurestaticapps.net`:

1. In the Azure portal, open the Static Web App resource and go to **Custom domains** → **Add**.
2. Enter `thefirstbookonllm.com`.
3. Azure shows you a CNAME or TXT record to add at your DNS registrar.
4. Add the record at the registrar (the apex domain `thefirstbookonllm.com` typically needs an `ALIAS`/`ANAME` record pointing to the SWA hostname; the `www` subdomain takes a CNAME).
5. Click **Validate**. Once DNS propagates (a few minutes to a few hours), Azure auto-provisions a free TLS certificate.
6. Set the apex `thefirstbookonllm.com` as the default and `www.thefirstbookonllm.com` as a redirect alias (or vice-versa) under **Custom domains**.

If your registrar does not support `ALIAS`/`ANAME` records on the apex, point `www.thefirstbookonllm.com` at SWA via CNAME and set up a registrar-side redirect from the apex to `www`.

## Updating content

- Copy edits, link changes, new sections → edit `index.html`.
- Visual changes → edit `styles.css`.
- Replacing or adding diagrams → drop new SVGs into `diagrams/` and reference them from the `<section id="peek">` block. Source SVGs live in the parent repo at `../svg/`.
- New pricing or buy links → edit the two `volume-buy` blocks and the `buy-grid` near the bottom of `index.html`.
- Author bio → edit the `<section id="author">` block in `index.html`. Headshot is `ritesh.jpeg`. Add or change role pills inside `.author-roles`. The two `<p class="author-text">` paragraphs are conservative on purpose. Replace with the full bio (publications, talks, awards, etc.) once it is finalized.

After committing changes, the GitHub Actions workflow Azure created on first deploy will redeploy automatically.

## What is on the site

- **Hero** with the three LinkedIn questions (duplicate prompt, strawberry, LoRA on one GPU) and a positioning paragraph that mirrors `promotion.md`.
- **Why this book** with four cards (one per book color) explaining the mechanism behind a trick the reader has already noticed.
- **The two volumes** with cover mockups, topic lists, and direct buy buttons.
  - Volume I: Leanpub + Amazon.
  - Volume II: Amazon.
- **A peek inside** with seven actual diagrams from the book chapters, plus the line *"this is not even one percent of what is in there."*
- **Outcomes** — seven concrete things the reader can do after both volumes.
- **Final buy section** with both volumes and an italic close.

No tracking, no third-party JS, no analytics by default. Add Plausible / GA / Azure Application Insights later if you want measurement.

## Hardening before launch

- Verify all four buy links open in a new tab and resolve to the correct product page.
- Cover images are 2400x3000 JPEGs (~430 KB each). Consider generating WebP versions and using `<picture>` with `<source type="image/webp">` for ~70% smaller transfer on supporting browsers. Optional but a quick win for Lighthouse.
- Run a Lighthouse audit (Performance, Accessibility, Best Practices, SEO) and fix anything below 95. The Vol I cover is loaded with `fetchpriority="high"` to keep LCP fast; Vol II and the mini covers are lazy-loaded.
- Add a privacy / contact page if you collect any reader data later (currently the site collects nothing).

## SEO setup (rock-solid)

The site is built for crawlability, structured data, and Core Web Vitals.

**Structured data (JSON-LD `@graph` in `index.html`)**
- `WebSite` with publisher reference.
- `Person` for Ritesh Modi (`worksFor: MarketOnce`, `alumniOf: Microsoft`, `sameAs: riteshmodi.com`).
- `Book` for Volume I with two `Offer` nodes (Leanpub + Amazon) and topical `about` keywords.
- `Book` for Volume II with one `Offer` (Amazon) and topical `about` keywords.
- `FAQPage` with eight `Question`/`Answer` pairs that mirror the visible FAQ section verbatim.

Validate with [Google Rich Results Test](https://search.google.com/test/rich-results) and [Schema.org Validator](https://validator.schema.org/) before launch. Both should report Book + FAQ + Person + WebSite without errors.

**Meta + social**
- `<title>` is 53 characters, ideal range 50–60.
- `<meta name="description">` is ~155 characters, ideal range 150–160.
- `<meta name="robots">` explicitly sets `index, follow, max-image-preview:large`.
- Open Graph tags fully populated (title, description, image with dimensions and alt, secure_url, locale, site_name, book:author).
- Twitter Card is `summary_large_image` with creator handle.
- Update the `twitter:creator` handle in `index.html` if `@riteshmodi` is wrong.

**Crawlability**
- `robots.txt` explicitly allows Googlebot, Bingbot, DuckDuckBot, Slurp, facebookexternalhit, Twitterbot, LinkedInBot, GPTBot, ClaudeBot, PerplexityBot, Google-Extended.
- `sitemap.xml` lists the canonical URL and three sibling images (Vol I cover, Vol II cover, author headshot).
- `<link rel="sitemap">` and `<link rel="manifest">` declared in `<head>`.

**Performance hints**
- `<link rel="preload" as="image" fetchpriority="high">` for Vol I cover (LCP candidate).
- `<link rel="preconnect">` to `fonts.googleapis.com` and `fonts.gstatic.com`.
- `<link rel="dns-prefetch">` to `leanpub.com` and `amazon.com` (the buy-button targets).
- `staticwebapp.config.json` sets `Cache-Control: max-age=31536000, immutable` for all images and CSS, and `max-age=600` for the HTML so updates ship promptly.
- Vol II cover and mini covers use `loading="lazy"`.

**Accessibility (impacts SEO)**
- `<a class="skip-link">` keyboard-visible skip-to-content target.
- Single H1, hierarchical H2 / H3.
- All `<img>` carry `alt` text and explicit `width`/`height` to prevent CLS.
- `:focus-visible` styles applied site-wide.
- `prefers-reduced-motion` respected.
- `<nav aria-label="Primary">` and HTML5 landmarks (`header`, `main`, `footer`) for screen readers.

**Security headers (lift Lighthouse Best Practices)**
- HSTS with `preload` (set up the [HSTS preload list](https://hstspreload.org/) submission after launch confirms).
- `X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `Referrer-Policy: strict-origin-when-cross-origin`.
- `Permissions-Policy` denies camera/microphone/geolocation/FLoC.
- `Content-Security-Policy` allowlists self + Google Fonts only. If you add analytics or ads later, extend the CSP.

**Post-launch checklist**
1. Submit `https://thefirstbookonllm.com/` to [Google Search Console](https://search.google.com/search-console). Verify ownership via DNS TXT record on the same registrar where DNS lives.
2. Submit the sitemap inside Search Console: *Sitemaps* → enter `sitemap.xml`.
3. Submit to [Bing Webmaster Tools](https://www.bing.com/webmasters).
4. Run [PageSpeed Insights](https://pagespeed.web.dev/) on the live URL. Target: 95+ on all four scores. Fix any LCP / CLS / INP regressions.
5. Run the [Mobile-Friendly Test](https://search.google.com/test/mobile-friendly).
6. Test the OG card with [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/), [Twitter Card Validator](https://cards-dev.twitter.com/validator), and [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/). Force a re-scrape if the cached card is stale.
7. After two weeks of indexing, verify in Search Console that Book and FAQ rich results are being recognized. Add author Knowledge Panel claim via [Google Knowledge Panel](https://support.google.com/knowledgepanel).
