# Deploying hornbeckco.com — GitHub Pages runbook

This folder is a ready git repo (branch `main`, one commit) with `CNAME` and `.nojekyll` already in place. `specimen.pdf` and `data-memo.pdf` sit at the exact root paths the five staged cold-email drafts link to. No build step, no dependencies.

**Preview locally:** `python3 -m http.server 8080` in this folder → http://localhost:8080

Note: GitHub Pages on the free plan requires a **public** repo (private-repo Pages needs GitHub Pro). This is a marketing site — public source is fine; that's what hosting it means anyway.

## 1 · Push the repo (~2 min)

With gh CLI:

    cd hornbeckco_site
    gh repo create hornbeckco-site --public --source=. --push

Or web UI: create an empty public repo `hornbeckco-site` on github.com, then:

    git remote add origin https://github.com/<YOUR-USER>/hornbeckco-site.git
    git push -u origin main

Commit author is set to Hornbeckco <hello@hornbeckco.com>; amend if you care.

## 2 · Turn on Pages (~1 min)

Repo → Settings → Pages → Build and deployment → Source: **Deploy from a branch** → `main` / `/ (root)` → Save.
Site goes live at `https://<YOUR-USER>.github.io/hornbeckco-site/` within a minute — click through everything, including both PDFs and dark mode.

## 3 · Custom domain, in this order (protects against takeover + gets one cert for apex AND www)

1. **Verify the domain** (recommended, 5 min): github.com → Settings → Pages → "Add a domain" → `hornbeckco.com` → add the `_github-pages-challenge-<user>` TXT record it gives you at Namecheap → Verify.
2. Repo → Settings → Pages → **Custom domain**: `hornbeckco.com` → Save. (The CNAME file in the repo already matches, so nothing new is committed.)
3. **Namecheap → Domain List → hornbeckco.com → Advanced DNS.** Delete only the address records currently pointing `@` and `www` at Namecheap hosting. **Do not touch MX or any TXT record** — SPF, DKIM (`google._domainkey`), DMARC (`_dmarc`), and Google site verification are your email. Then add:

    | Type  | Host | Value                      |
    |-------|------|----------------------------|
    | A     | @    | 185.199.108.153            |
    | A     | @    | 185.199.109.153            |
    | A     | @    | 185.199.110.153            |
    | A     | @    | 185.199.111.153            |
    | CNAME | www  | <YOUR-USER>.github.io      |

   TTL: Automatic. Add the A records and the www CNAME in the same sitting — GitHub mints one certificate covering both, and a cert generated before both records exist has to be regenerated.
4. Back in repo → Settings → Pages: wait for the DNS check to go green, then tick **Enforce HTTPS** (cert usually provisions in minutes once DNS propagates; can take up to a day — another reason this happens Thursday, not Sunday).
5. **Confirmation:** in an incognito tab open https://hornbeckco.com, https://www.hornbeckco.com (should redirect), /specimen.pdf, /data-memo.pdf. Send + receive one email from hello@ to prove the mail records were untouched.

## 4 · After it's live

- Namecheap Stellar: remove the hornbeckco.com addon-domain hosting — inert once the A records moved. Registration and DNS stay at Namecheap; nameservers never changed.
- Google Search Console: the domain is already Google-verified for Workspace, so the property adds instantly; submit `sitemap.xml`.
- Every future edit: change files → `git commit` → `git push` → live in ~a minute. Claude Code sessions push directly. Updating the specimen later = replacing `specimen.pdf` and pushing.

## If you ever want Cloudflare instead

Same repo connects to Cloudflare Pages / Workers Static Assets in ~10 min (framework None, output `/`); that route moves nameservers to Cloudflare, so re-verify the four Workspace mail records survive the zone import before flipping. Nothing about the site itself changes.

## Edit notes

- **About section** (`index.html`, `#about`): currently "Aidan" + the CFP Board-Registered Program line — add your full name and any bio detail you want public. Same check applies to the five draft signatures.
- Homepage specimen figures (~13.7%, $128,957, $14,563→$4,819) are hard-coded from the shipped specimen; if the specimen is rebuilt, update both together.
- Brand tokens are in `:root` at the top of `style.css`; teal stays reserved for actions/links.
