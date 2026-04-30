# thefirstbookonllm.com — static website

Single-page landing site for *Inside Large Language Models*, Volumes I and II. Pure HTML and CSS. No build step. Deploys to Azure Static Web Apps in a few minutes.

## Files

```
website/
├── index.html                              main page
├── styles.css                              styles (book's four-color palette)
├── staticwebapp.config.json                Azure Static Web Apps config (routing, headers, caching)
├── Inside-LLMs-Vol1-Cover-2400x3000.jpg    Volume I cover
├── Inside-LLMs-Vol2-Cover-2400x3000.jpg    Volume II cover
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
