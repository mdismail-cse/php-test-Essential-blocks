# Essential Blocks — PHP Fatal Compatibility Workflow

Automated GitHub Actions workflow that tests whether **Essential Blocks (free)**, **Essential Blocks Pro**, and the **`src/controls` submodule** trigger a PHP fatal error or a PHP-version incompatibility on any supported PHP version, inside a real WordPress install.

> **Design rule:** a green report means *"we actually tested and found nothing."* If a test could not run (build / install / auth / server / scan failure), the leg is reported as **INCONCLUSIVE** — never as PASS. A real fatal turns the whole run **red**.

## 📋 Table of Contents

- [Overview](#overview)
- [What gets tested](#what-gets-tested)
- [Workflow architecture](#workflow-architecture)
- [Verdicts: how a result is decided](#verdicts-how-a-result-is-decided)
- [Prerequisites](#prerequisites)
- [Setup instructions](#setup-instructions)
- [Running the workflow](#running-the-workflow)
- [Understanding the report](#understanding-the-report)
- [Workflow steps explained](#workflow-steps-explained)
- [Error handling philosophy](#error-handling-philosophy)
- [Troubleshooting](#troubleshooting)
- [Technical details](#technical-details)
- [Configuration options](#configuration-options)

---

## 🎯 Overview

This workflow performs, for every PHP version in the matrix:

- **Plugin builds** — builds `src/controls`, then free, then Pro using pnpm.
- **Static compatibility scan** — PHPCompatibilityWP analysis of the free and Pro PHP code against the target PHP version.
- **Real WordPress install** — downloads WordPress, sets up MySQL, installs and activates both plugins.
- **Runtime fatal detection** — logs in via a headless browser and loads admin + front-end pages, flagging HTTP 500 (white screen of death) or fatal-error markers in the HTML.
- **Debug-log scan** — captures `wp-content/debug.log` and counts `PHP Fatal error` entries.
- **Trustworthy reporting** — a combined HTML + PDF report with a **PASS / FAIL / INCONCLUSIVE** verdict per PHP version, deployed to GitHub Pages.

There are **three independent fatal signals** (static scan, runtime HTTP/HTML, debug.log) so a real fatal is hard to miss, and "we couldn't test" is never reported as a pass.

---

## 🧪 What gets tested

| Repo | How it's tested |
|------|-----------------|
| `WPDevelopers/essential-blocks` (free) | Built with pnpm, copied into WordPress, scanned and exercised at runtime. The `controls` submodule is built into the free plugin, so its PHP surfaces here. |
| `WPDevelopers/essential-blocks-pro` (Pro) | Built with pnpm, copied into WordPress, scanned and exercised at runtime alongside the free plugin. |
| `EssentialBlocks/controls` (`src/controls` submodule) | Checked out to the requested branch and built first. It is primarily JavaScript; its PHP impact is covered through the free-plugin scan. |

Each run lets you pick the branch for all three independently (`branch`, `pro_branch`, `controls_branch`).

---

## 🏗️ Workflow architecture

```
┌───────────────────────────────────────────────────────────────┐
│                     GitHub Actions Workflow                     │
│   "WP PHP Fatal Compatibility (free + pro + controls × PHP)"    │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│  Job 1: phpcompat   (matrix: 8.5, 8.4, 8.3, 8.2, 8.1, 8.0, 7.4)│
│                     (fail-fast: false · timeout: 45 min)        │
├───────────────────────────────────────────────────────────────┤
│   1. Checkout free (+ submodules) and Pro                       │
│   2. Initialize per-leg summary.env (defaults = "untested")     │
│   3. pnpm + Node.js 22 + Puppeteer cache                        │
│   4. Build controls → free → Pro  (records BUILD_* statuses)    │
│   5. Setup PHP (matrix) + Composer + WP-CLI                     │
│   6. Install MySQL + PHPCompatibilityWP                         │
│   7. Prepare + install WordPress       (records WP_INSTALL)     │
│   8. Copy (rsync) + activate plugins   (records PLUGINS_ACTIVE) │
│   9. Start WP server (multi-worker)    (records SERVER)         │
│  10. PHPCompatibility scan (JSON)      (records PHPCOMPAT_*)    │
│  11. Runtime fatal check + screenshots (records AUTH, RUNTIME_*)│
│  12. Save + scan debug.log             (records DEBUGLOG_FATALS)│
│  13. Upload per-PHP artifact (always)                           │
└───────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────────┐
│  Job 2: combine     (needs: phpcompat · if: always())          │
├───────────────────────────────────────────────────────────────┤
│   1. Download all per-PHP artifacts                             │
│   2. Build combined HTML + compute PASS/FAIL/INCONCLUSIVE       │
│      verdict from each leg's summary.env                        │
│   3. Generate PDF (headless Chromium)                           │
│   4. Upload final artifact + deploy to GitHub Pages            │
│   5. Enforce verdict: fail the run if any leg is FAIL or if     │
│      zero legs reported                                         │
└───────────────────────────────────────────────────────────────┘
```

---

## ⚖️ Verdicts: how a result is decided

Each PHP leg writes a machine-readable `summary.env`. The `combine` job reads it and assigns one verdict:

| Verdict | When | Meaning |
|---------|------|---------|
| 🟥 **FAIL** | `PHPCOMPAT_ERRORS > 0` **or** `RUNTIME_FATALS > 0` **or** `DEBUGLOG_FATALS > 0` | A real PHP fatal or version incompatibility was found. |
| 🟨 **INCONCLUSIVE** | `WP_INSTALL ≠ ok`, `SERVER ≠ ok`, `AUTH ≠ ok`, `PHPCOMPAT_RAN ≠ yes`, `PLUGINS_ACTIVE = fail`, `RUNTIME_INCONCLUSIVE > 0`, **or** the leg never reported | The environment could not be tested reliably. **Do not treat as a pass.** |
| 🟩 **PASS** | none of the above | Tested, no fatal/incompatibility found. |

FAIL takes priority over INCONCLUSIVE — a found fatal is reported even if some other check was shaky. The whole workflow run fails (red ✗) if **any** leg is FAIL, or if **no** legs reported results (e.g. a bad PAT made every checkout fail).

`summary.env` keys: `PHP`, `FREE_BRANCH`, `PRO_BRANCH`, `CONTROLS_BRANCH`, `BUILD_CONTROLS`, `BUILD_FREE`, `BUILD_PRO`, `WP_INSTALL`, `PLUGINS_ACTIVE`, `SERVER`, `AUTH`, `PHPCOMPAT_RAN`, `PHPCOMPAT_ERRORS`, `PHPCOMPAT_WARNINGS`, `RUNTIME_PAGES`, `RUNTIME_FATALS`, `RUNTIME_INCONCLUSIVE`, `DEBUGLOG_FATALS`.

---

## 📦 Prerequisites

### Required GitHub secret

| Secret | Description |
|--------|-------------|
| `ESSENTIAL_BLOCKS_PAT` | Personal Access Token with **read access to both private repos**: `WPDevelopers/essential-blocks` *and* `WPDevelopers/essential-blocks-pro`. Use the `repo` scope (classic) or contents:read on both repos (fine-grained). |

**To add it:** repository → **Settings → Secrets and variables → Actions → New repository secret**, name `ESSENTIAL_BLOCKS_PAT`.

> If the PAT lacks access to *either* repo (or expires), every leg fails at checkout and the run fails loudly with "No PHP legs reported any results."

### Repository access

The PAT must be able to read:
- `WPDevelopers/essential-blocks` (free)
- `WPDevelopers/essential-blocks-pro` (Pro)
- `EssentialBlocks/controls` is pulled as a submodule of the free repo.

### GitHub Pages (optional)

To publish the report, enable Pages from the `gh-pages` branch — see [GITHUB_PAGES_SETUP.md](GITHUB_PAGES_SETUP.md).

---

## 🚀 Setup instructions

1. **Add the secret** — `ESSENTIAL_BLOCKS_PAT` as above.
2. **(Optional) adjust the PHP matrix** — edit [.github/workflows/wp-phpcompat-full.yml](.github/workflows/wp-phpcompat-full.yml):
   ```yaml
   strategy:
     matrix:
       # Quote each value so 8.0/8.5 aren't parsed as floats.
       php: ['8.5', '8.4', '8.3', '8.2', '8.1', '8.0', '7.4']
   ```
3. **(Optional) enable GitHub Pages** — see [GITHUB_PAGES_SETUP.md](GITHUB_PAGES_SETUP.md).

---

## ▶️ Running the workflow

This workflow is **manual-only** (`workflow_dispatch`).

### Via GitHub UI

1. **Actions** tab → **WP PHP Fatal Compatibility (free + pro + controls × all PHP versions)**.
2. **Run workflow**, then set the branches:
   - **branch** — Essential Blocks (free) branch (default `master`)
   - **pro_branch** — Essential Blocks Pro branch (default `main`)
   - **controls_branch** — `src/controls` submodule branch (default `master`)
3. **Run workflow**.

### Via GitHub CLI

```bash
gh workflow run "WP PHP Fatal Compatibility (free + pro + controls × all PHP versions)" \
  -f branch=master \
  -f pro_branch=main \
  -f controls_branch=master
```

> Tip: you can also target it by file name: `gh workflow run wp-phpcompat-full.yml -f branch=master`.

A `concurrency` group cancels an in-progress run on the same ref when you start a new one, so two dispatches never race on the `gh-pages` branch.

---

## 📊 Understanding the report

### Where to find it

- **GitHub Pages** (if enabled): `https://<user>.github.io/<repo>/` — always the latest run.
- **Artifacts**: the workflow run page → **Artifacts** → `essential-blocks-wp-compat-final` (HTML + PDF + screenshots, 30-day retention). Per-PHP raw logs are in `perphp-reports-<php>` (7-day retention).

### Final artifact contents

```
essential-blocks-wp-compat-final/
├── combined.html      # Main report (deployed as index.html on Pages)
├── combined.pdf       # PDF render of the report
└── *.jpg              # Screenshots from every PHP version
```

### Summary table columns

| Column | Description |
|--------|-------------|
| **PHP** | PHP version tested |
| **Verdict** | PASS / FAIL / INCONCLUSIVE (see [Verdicts](#verdicts-how-a-result-is-decided)) |
| **PHPCompat errors** | PHPCompatibility static-scan error count |
| **Runtime fatals** | Pages that returned HTTP 500 or contained a fatal marker |
| **debug.log fatals** | `PHP Fatal error` lines found in `wp-content/debug.log` |
| **Why** | One-line explanation of the verdict |

### Status badges

| Badge | Meaning |
|-------|---------|
| 🟩 **PASS** | Tested, no fatal/incompatibility found |
| 🟥 **FAIL** | A real PHP fatal or version incompatibility was found |
| 🟨 **INCONCLUSIVE** | Could not be reliably tested — do **not** treat as a pass |

### Per-version detail sections

For each PHP version the report shows (when present): the raw `summary.env`, the PHPCompatibility text report, the runtime fatal-check log, the WordPress `debug.log`, any setup errors, and the screenshots.

---

## 🔧 Workflow steps explained

### Checkout (free + Pro)

```yaml
- uses: actions/checkout@v4
  with:
    repository: WPDevelopers/essential-blocks
    ref: ${{ github.event.inputs.branch }}
    token: ${{ secrets.ESSENTIAL_BLOCKS_PAT }}
    path: essential-blocks
    submodules: recursive
# ...and a second checkout for essential-blocks-pro (ref: pro_branch)
```

Both private repos are cloned with the PAT; the free repo pulls `src/controls` recursively.

### Initialize per-leg summary

Creates `reports/<php>/summary.env` with every key defaulted to an "untested" value, plus the report subdirectories. This is what makes a non-running step show as INCONCLUSIVE instead of silently passing.

### pnpm + Node.js + Puppeteer cache

- `pnpm/action-setup@v4` (pnpm 9)
- `actions/setup-node@v4` (Node.js 22, pnpm cache keyed on `**/pnpm-lock.yaml`)
- Puppeteer's Chromium cached at `~/.cache/puppeteer`, keyed on `pnpm-lock.yaml`.

### Build controls → free → Pro

```bash
# inside essential-blocks:
git submodule update --init --recursive
# switch src/controls to the requested branch, then:
( cd src/controls && pnpm install && pnpm run build )   # BUILD_CONTROLS
pnpm install && pnpm run build                          # BUILD_FREE
# inside essential-blocks-pro:
pnpm install && pnpm run build                          # BUILD_PRO
```

Each build's real exit status is recorded (`ok`/`fail`) in `summary.env`. The build logs land in `reports/<php>/build/`. A JS build failure does **not** suppress the PHP static scan (which works on source regardless).

### Setup PHP + tools

```yaml
- uses: shivammathur/setup-php@v2
  with:
    php-version: ${{ matrix.php }}
    tools: composer, wp-cli
    extensions: mbstring, intl, curl, zip, xml, json, mysqli
```

`phpcs` / PHPCompatibilityWP are installed separately via `composer global require` and added to `$GITHUB_PATH`.

### Prepare + install WordPress

- Starts MySQL; for **PHP < 8.0** (numeric `sort -V` check) switches the root user to `mysql_native_password` so `mysqli` can connect to MySQL 8.
- Creates the DB, downloads WordPress, writes `wp-config.php` with `WP_DEBUG`, `WP_DEBUG_LOG`, and `WP_DEBUG_DISPLAY=false` (fatals go to the log / produce an HTTP 500, keeping screenshots clean).
- Installs WordPress and verifies it responds (`wp option get siteurl`) → records `WP_INSTALL`.

### Copy + activate plugins

```bash
rsync -a --exclude node_modules --exclude .git essential-blocks/     wp-content/plugins/essential-blocks/
rsync -a --exclude node_modules --exclude .git essential-blocks-pro/ wp-content/plugins/essential-blocks-pro/
wp plugin activate essential-blocks essential-blocks-pro --allow-root
```

`node_modules`/`.git` are excluded (large/irrelevant) but `vendor` is **kept** (Composer autoload is needed at runtime). Activation result → `PLUGINS_ACTIVE` (`ok`/`partial`/`fail`).

### Start WordPress server (multi-worker)

```bash
PHP_CLI_SERVER_WORKERS=8 nohup wp server --host=127.0.0.1 --port=8080 --path=. --allow-root &
```

`PHP_CLI_SERVER_WORKERS` lets the PHP built-in server handle the concurrent sub-requests (admin-ajax / REST / assets) an admin page makes, instead of deadlocking. The step waits for the server and records `SERVER`.

### PHPCompatibility scan

```bash
phpcs --standard=PHPCompatibilityWP \
      --runtime-set testVersion "${{ matrix.php }}" \
      --extensions=php --ignore='*/node_modules/*,*/.pnpm/*' \
      --report-full=...txt --report-json=...json \
      wp-content/plugins/essential-blocks \
      wp-content/plugins/essential-blocks-pro
```

Both plugins are scanned in one pass. Error/warning **counts are parsed from the JSON report** (authoritative `totals`), not by counting log lines. Records `PHPCOMPAT_RAN`, `PHPCOMPAT_ERRORS`, `PHPCOMPAT_WARNINGS`.

### Runtime fatal check + screenshots

A single headless-browser (Puppeteer) session logs in with the admin password, then for each target page records a verdict (`OK` / `FATAL` / `INCONCLUSIVE`) and saves a 1920×1080 JPEG screenshot. A page is **FATAL** if it returns HTTP ≥ 500 or its HTML contains a fatal/parse/uncaught marker; a lost session is **INCONCLUSIVE**, never a silent pass.

Pages checked:

| Name | URL |
|------|-----|
| `frontend_home` | `/` |
| `dashboard` | `/wp-admin/` |
| `plugins` | `/wp-admin/plugins.php` |
| `eb_options` | `/wp-admin/admin.php?page=essential-blocks&tab=options` |
| `eb_quick_setup` | `/wp-admin/admin.php?page=eb-quick-setup` (Pro deactivated around this free-only screen) |
| `editor_post` | `/wp-admin/post-new.php?post_type=post` |
| `editor_page` | `/wp-admin/post-new.php?post_type=page` |

Records `AUTH`, `RUNTIME_PAGES`, `RUNTIME_FATALS`, `RUNTIME_INCONCLUSIVE`.

### Save + scan debug.log

Copies `wp-content/debug.log` to the report and counts `PHP Fatal error` lines → `DEBUGLOG_FATALS`.

### Combine + deploy

The `combine` job downloads every artifact, computes the verdict table, renders `combined.html`, prints it to `combined.pdf` with headless Chromium, uploads the final artifact, and deploys `index.html` + screenshots + PDF to GitHub Pages. A final step fails the run if any leg is FAIL or if no legs reported.

---

## 🛡️ Error handling philosophy

This workflow is **not** "best effort, always green." It deliberately separates two kinds of failure:

- **A real fatal / incompatibility** → the leg is **FAIL** and the whole run goes **red**.
- **An environment that couldn't be tested** (build/install/auth/server/scan gap) → the leg is **INCONCLUSIVE**, surfaced clearly, and never counted as a pass.

Individual setup commands still tee their output to per-step logs so you can debug, but their *status* is recorded and rolled up into the verdict rather than being swallowed. The artifact upload uses `if: always()`, so even a leg that dies mid-way still publishes whatever it captured.

### Per-version artifact layout

```
reports/<php>/
├── summary.env                 # machine-readable status (drives the verdict)
├── setup/
│   ├── errors.log
│   ├── mysql-config.log
│   ├── db-create.log
│   ├── wp-download.log
│   ├── wp-config.log
│   ├── wp-install.log
│   ├── plugin-copy.log / pro-plugin-copy.log
│   └── plugin-activate.log / pro-plugin-activate.log
├── build/
│   ├── submodule.log
│   ├── controls-branch.log
│   ├── controls-install.log / controls-build.log
│   ├── free-install.log / free-build.log
│   └── pro-install.log / pro-build.log
├── phpcompat/
│   ├── phpcompat-<php>.txt      # human-readable report
│   └── phpcompat-<php>.json     # authoritative counts
├── fatal/
│   └── runtime-<php>.log        # per-page VERDICT lines
├── wpdebug/
│   └── debug-<php>.log
└── screens/
    └── <php>_<page>.jpg
```

---

## 🔍 Troubleshooting

#### Run fails immediately on every PHP version
**Cause:** `ESSENTIAL_BLOCKS_PAT` is missing, expired, or lacks access to one of the private repos.
**Fix:** confirm the secret exists and can read both `essential-blocks` and `essential-blocks-pro`. The run will say *"No PHP legs reported any results."*

#### A leg shows INCONCLUSIVE
**Cause:** WordPress install, plugin activation, server start, login, or the static scan couldn't complete. **It does not mean "passed."**
**Fix:** open that version's section in the report — the **Why** column and the `setup/errors.log` / `runtime-<php>.log` pinpoint which step failed. Re-run after fixing the environment issue.

#### A leg shows FAIL
**Cause:** a real PHP fatal or version incompatibility. Check the **PHPCompatibility** report (static), the **Runtime fatal check** log (which page 500'd), and the **debug.log** for the stack trace.

#### A new PHP version (e.g. 8.5) is INCONCLUSIVE while older ones pass
**Possible causes:** PHPCompatibilityWP may not yet ship full rule coverage for the newest PHP (the runtime + debug.log signals still apply), or `composer global require` couldn't satisfy a platform constraint under that PHP. For the latter, adding `--ignore-platform-req=php` to the Composer step lets the tools install.
**Note:** on a brand-new PHP, a dashboard HTTP 500 can be WordPress core itself, not the plugins — the static scan (scoped to the plugin dirs) is the cleaner per-repo signal there.

#### Screenshots show the login page
**Cause:** authentication didn't hold. The affected pages are reported as INCONCLUSIVE (not OK), and `AUTH` is recorded in `summary.env`. Check `wp-install.log` and the runtime log.

#### PDF not generated
**Cause:** the headless-Chromium print step failed. The HTML report is still produced and deployed; the PDF is best-effort.

---

## 🔬 Technical details

### PHP versions tested

| Version | Notes |
|---------|-------|
| 8.5 | Newest; static-scan rule coverage may lag — rely on runtime + debug.log signals |
| 8.4 | |
| 8.3 | |
| 8.2 | |
| 8.1 | |
| 8.0 | |
| 7.4 | Minimum supported; uses `mysql_native_password` for MySQL 8 |

### Tools & dependencies

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 22 | Build the plugins; run Puppeteer |
| pnpm | 9 | Package manager |
| PHP | matrix | Target version under test |
| Composer | latest | Installs phpcs + PHPCompatibilityWP |
| WP-CLI | latest | WordPress setup |
| PHP_CodeSniffer | 3.x | Static-analysis framework |
| PHPCompatibilityWP | latest | PHP-version compatibility rules |
| Puppeteer (Chromium) | latest | Runtime fatal check, screenshots, **and** PDF rendering |
| MySQL | 8 | Database (installed via apt) |

### Runner & timing

- **OS:** `ubuntu-latest` · per-leg `timeout-minutes: 45`.
- **Matrix:** `fail-fast: false` (one failing version doesn't stop the others); `combine` runs with `if: always()`.
- All PHP versions run in parallel; wall-clock is roughly the slowest single leg plus the combine job.

---

## ⚙️ Configuration options

All edits are in [.github/workflows/wp-phpcompat-full.yml](.github/workflows/wp-phpcompat-full.yml).

### PHP versions

```yaml
strategy:
  matrix:
    php: ['8.5', '8.4', '8.3', '8.2', '8.1', '8.0', '7.4']  # keep values quoted
```

### Default branches

```yaml
on:
  workflow_dispatch:
    inputs:
      branch:          { default: 'master' }  # free
      pro_branch:      { default: 'main'   }  # pro
      controls_branch: { default: 'master' }  # src/controls
```

### Pages to check at runtime

Edit the `TARGETS` array in the **Runtime fatal check** step (`"<url> <name>"` per line).

### Artifact retention

```yaml
# per-PHP artifact
retention-days: 7
# final combined artifact
retention-days: 30
```

---

## 📄 License

MIT.

## 🙏 Acknowledgments

Essential Blocks team · PHPCompatibility · WordPress · GitHub Actions.

---

**Made with ❤️ for WordPress QA & compatibility testing**
