# 📄 GitHub Pages Setup for the Compatibility Report

This guide explains how to publish the PHP fatal-compatibility report to GitHub Pages so it's viewable at a public URL without downloading artifacts.

---

## 🎯 What this does

After each workflow run, the `combine` job deploys the report to GitHub Pages at:

```
https://<your-username>.github.io/<repo-name>/
```

**What gets deployed (to the `gh-pages` branch):**
- ✅ `combined.html` — published as `index.html`
- ✅ `combined.pdf` — the PDF render of the report
- ✅ `*.jpg` — all screenshots referenced by the report

**Behavior:**
- ✅ **Always latest** — each run replaces the previous deployment (`force_orphan: true`, `keep_files: false`).
- ✅ **Single stable URL** — bookmark it; it always shows the most recent run.
- ✅ **No history kept on Pages** — older reports remain available as workflow **artifacts** (30-day retention).

> Earlier versions of this workflow deployed only the HTML. The current workflow deploys the HTML, the PDF, and the screenshots together, so images render directly on the hosted page.

---

## 🔧 Setup instructions

### Step 1 — Enable GitHub Pages

1. Repository → **Settings → Pages**.
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. **Branch:** `gh-pages` · **Folder:** `/ (root)` → **Save**.

> Keep the source as **"Deploy from a branch → `gh-pages`"**. The workflow deploys via [`peaceiris/actions-gh-pages`](https://github.com/peaceiris/actions-gh-pages), which pushes to that branch. If you switch the source to "GitHub Actions", the pushed report will stop showing up.

The `gh-pages` branch is created automatically by the first successful run, so you may need to run the workflow once before the branch appears in the Pages dropdown.

### Step 2 — Run the workflow

1. **Actions** tab → **WP PHP Fatal Compatibility (free + pro + controls × all PHP versions)**.
2. **Run workflow**, choose the `branch` / `pro_branch` / `controls_branch`, and confirm.

### Step 3 — Wait for deployment

- The matrix + combine jobs finish, then GitHub Pages takes another 1–2 minutes to publish.
- You can watch it under **Settings → Pages** and in the **Deploy to GitHub Pages** step logs.

### Step 4 — Open your report

```
https://<your-username>.github.io/<repo-name>/
```

Example: `https://mdismail-cse.github.io/php-test-Essential-blocks/`

---

## 🔍 How it works

1. The `combine` job builds `final/combined.html`, renders `final/combined.pdf` (headless Chromium), and gathers the screenshots into `final/`.
2. The **Prepare GitHub Pages** step assembles `gh-pages-deploy/`:
   ```
   gh-pages-deploy/
   ├── .nojekyll          # disables Jekyll processing
   ├── index.html         # copied from combined.html
   ├── combined.pdf
   └── *.jpg              # screenshots
   ```
3. The **Deploy to GitHub Pages** step force-pushes that directory to the `gh-pages` branch:
   ```yaml
   - uses: peaceiris/actions-gh-pages@v3
     with:
       github_token: ${{ secrets.GITHUB_TOKEN }}
       publish_dir: ./gh-pages-deploy
       keep_files: false
       force_orphan: true
   ```
4. GitHub Pages serves it over HTTPS.

A `concurrency` group on the workflow cancels an in-progress run on the same ref when a new one starts, so two runs never race to push `gh-pages`.

---

## 🛠️ Troubleshooting

### 404 — page not found
- Ensure **Settings → Pages → Source** is **Deploy from a branch → `gh-pages`**.
- Make sure at least one run finished successfully (that's what creates the `gh-pages` branch).
- Wait 1–2 minutes after the run for Pages to build.

### The page is stale / not updating
- Confirm the latest run's **Deploy to GitHub Pages** step succeeded.
- Check the Pages source is still `gh-pages` (not "GitHub Actions").
- A failed `combine` job (e.g. an earlier step errored) means nothing new was deployed.

### Images don't load
- Screenshots **are** deployed now. If they're missing, the runtime step produced no screenshots for that run — check the per-PHP `screens/` folder in the artifact and the runtime log.

### Need an older report
- Old reports aren't kept on Pages. Download the `essential-blocks-wp-compat-final` artifact from the relevant workflow run (kept 30 days).

### PDF missing on the page
- PDF rendering is best-effort (headless Chromium). The HTML report still deploys even if the PDF step fails.

---

## 🔒 Security considerations

- **Public repo:** the report (including screenshots) is publicly accessible at the Pages URL.
- **Private repo:** GitHub Pages for private repos requires the appropriate plan; otherwise prefer artifacts-only.
- The report contains admin-area screenshots and logs from a throwaway test site (admin/admin on `127.0.0.1`); it does not expose production data, but treat the URL as you would any internal QA dashboard.

To avoid hosting entirely, remove the **Prepare GitHub Pages** and **Deploy to GitHub Pages** steps and rely on the `essential-blocks-wp-compat-final` artifact.

---

## 📊 Pages vs. artifacts

| | Artifacts | GitHub Pages |
|---|-----------|--------------|
| **Access** | Download + unzip | Direct browser URL |
| **Sharing** | Send files | Send a link |
| **History** | 30 days, every run | Latest run only |
| **Contents** | Everything (incl. per-PHP raw logs) | HTML + PDF + screenshots |

---

**Questions?** Check the **Deploy to GitHub Pages** step logs in the Actions run.
