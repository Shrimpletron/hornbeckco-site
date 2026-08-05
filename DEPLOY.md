# Deploying hornbeckco.com — runbook

Everything in this folder is the complete site. No build step, no dependencies, no external requests. `specimen.pdf` and `data-memo.pdf` are already in place at the exact paths the five staged cold-email drafts link to.

**Preview locally first:** `python3 -m http.server 8080` in this folder → http://localhost:8080

---

## Route A — fastest live (no repo): Cloudflare direct upload, ~10 min

1. dash.cloudflare.com → Workers & Pages → Create → Pages → **Upload assets** (direct upload).
2. Project name `hornbeckco`, drag this folder in, Deploy. You're live at `hornbeckco.pages.dev` immediately — click through every link and both PDFs.
3. Then do the custom-domain + DNS section below.

## Route B — the durable workflow (recommended): GitHub → Cloudflare, ~20 min

1. `git init && git add -A && git commit -m "hornbeckco.com v1"` → push to a new **private** GitHub repo `hornbeckco-site`.
2. Cloudflare dash → Workers & Pages → Create → Pages → **Connect to Git** → pick the repo.
   Build settings: framework **None**, build command **empty**, output directory **/** (root). Deploy.
3. From now on every push deploys automatically — Claude Code edits the site, `git push`, done. Updating the specimen later = replacing `specimen.pdf` in the repo.

*(Cloudflare now steers new projects toward "Workers → Static Assets" — same free tier, same steps, either surface is fine. Pages remains fully supported.)*

## Custom domain + the DNS move (the part that touches email — do it in this order)

1. Cloudflare dash → Add a site → `hornbeckco.com` → Free plan. Cloudflare scans and imports your existing Namecheap DNS records.
2. **STOP AND VERIFY before touching nameservers.** In the imported zone, confirm all Workspace records survived:
   - MX → `smtp.google.com` (priority 1)
   - TXT SPF → `v=spf1 include:_spf.google.com ~all`
   - TXT DKIM → `google._domainkey` (the long key from Admin console)
   - TXT DMARC → `_dmarc` → `v=DMARC1; p=none; rua=mailto:hello@hornbeckco.com`
   - The Google site-verification TXT, if present
   Missing any → add manually before proceeding. Set MX/TXT to "DNS only" (they are by default).
3. Namecheap → Domain List → hornbeckco.com → Nameservers → **Custom DNS** → paste the two Cloudflare nameservers. Propagation: minutes to a few hours; identical records = zero mail interruption *if step 2 was done*.
4. Back in the Pages project → Custom domains → add `hornbeckco.com` and `www.hornbeckco.com`. Cloudflare creates the records and provisions HTTPS automatically (the HTTPS checklist item is now permanently dead).
5. **Friday confirmation:** send a mail from hello@ to an outside address and reply to it; open https://hornbeckco.com/specimen.pdf and /data-memo.pdf in incognito. Green on both = Monday is clear.

## After it's live

- Namecheap Stellar: remove the hornbeckco.com addon-domain hosting whenever — it's inert once NS point at Cloudflare. Registration stays at Namecheap.
- Google Search Console: add the domain property (DNS verification is one click now that you're on CF), submit `sitemap.xml`.
- Test dark mode (the site ships light/dark via `prefers-color-scheme`).

## Edit notes

- **About section** (`index.html`, `#about`): currently "Aidan" with the CFP Board-Registered Program line — add your full name and any bio detail you want public. Same check applies to the five draft signatures.
- Specimen figures on the homepage (~13.7%, $128,957, $14,563→$4,819) are hard-coded from the shipped specimen. If the specimen is ever rebuilt, update both together.
- Brand tokens live at the top of `style.css` (`:root`); the accent teal is reserved for actions/links — keep it that way.
