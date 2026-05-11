# Essential Blocks - PHP Compatibility Testing Workflow

Automated PHP compatibility testing workflow for the Essential Blocks WordPress plugin across multiple PHP versions.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Workflow Architecture](#workflow-architecture)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Running the Workflow](#running-the-workflow)
- [Understanding the Reports](#understanding-the-reports)
- [Workflow Steps Explained](#workflow-steps-explained)
- [Error Handling](#error-handling)
- [Troubleshooting](#troubleshooting)
- [Technical Details](#technical-details)

---

## 🎯 Overview

This GitHub Actions workflow automatically tests the Essential Blocks WordPress plugin for PHP compatibility issues across multiple PHP versions. It performs:

- **Plugin Build** - Builds the plugin with all dependencies using pnpm
- **PHPCompatibility Scanning** - Static code analysis for PHP version compatibility
- **WordPress Installation** - Sets up a complete WordPress environment
- **Fatal Error Detection** - Checks admin pages for runtime errors
- **Visual Testing** - Captures screenshots of admin pages
- **Comprehensive Reporting** - Generates HTML and PDF reports with all findings

---

## ✨ Features

### 🔍 Comprehensive Testing
- ✅ Tests across multiple PHP versions (7.4, 8.0, 8.1, 8.2, 8.3, 8.4)
- ✅ Static code analysis with PHPCompatibilityWP
- ✅ Runtime fatal error detection on admin pages
- ✅ WordPress debug log capture
- ✅ Visual regression testing with screenshots

### 🔨 Plugin Build Process
- ✅ Automatic git submodule initialization
- ✅ Builds `src/controls` submodule first
- ✅ Builds root project with all dependencies
- ✅ Uses pnpm for fast, efficient package management
- ✅ Logs all build steps for debugging

### 📊 Advanced Reporting
- ✅ Combined HTML report with summary table
- ✅ Individual error reports per PHP version
- ✅ Color-coded status badges (PASS/FAIL/WARNING)
- ✅ Expandable sections for detailed logs
- ✅ Screenshots embedded in reports
- ✅ PDF export for easy sharing

### 🛡️ Error Handling
- ✅ Workflow continues even if steps fail
- ✅ All errors logged to dedicated files
- ✅ Setup errors reported separately
- ✅ Build errors tracked and displayed
- ✅ MySQL authentication compatibility for PHP < 7.4

---

## 🏗️ Workflow Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Actions Workflow                   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Job 1: phpcompat (Matrix: PHP 7.2, 7.4, 8.0, 8.1, 8.2...)  │
├─────────────────────────────────────────────────────────────┤
│  1. Checkout Essential Blocks (with submodules)              │
│  2. Setup Node.js 18 + pnpm                                  │
│  3. Build Plugin                                             │
│     ├── Initialize submodules                                │
│     ├── Build src/controls                                   │
│     └── Build root project                                   │
│  4. Setup PHP + Composer + WP-CLI                            │
│  5. Install PHPCompatibilityWP                               │
│  6. Setup MySQL + WordPress                                  │
│  7. Install WordPress                                        │
│  8. Copy & Activate Plugin                                   │
│  9. Start WordPress Server                                   │
│ 10. Run PHPCompatibility Scan                                │
│ 11. Check Admin Pages for Fatal Errors                       │
│ 12. Take Screenshots with Puppeteer                          │
│ 13. Upload Artifacts                                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Job 2: combine (Runs after all PHP versions complete)      │
├─────────────────────────────────────────────────────────────┤
│  1. Download all artifacts                                   │
│  2. Build combined HTML report                               │
│     ├── Summary table with all PHP versions                  │
│     ├── Detailed results per version                         │
│     ├── Individual error reports                             │
│     └── Embedded screenshots                                 │
│  3. Generate PDF from HTML                                   │
│  4. Upload final artifact                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Prerequisites

### Required GitHub Secrets

You need to configure the following secret in your GitHub repository:

| Secret Name | Description | How to Get |
|-------------|-------------|------------|
| `ESSENTIAL_BLOCKS_PAT` | Personal Access Token for accessing the private Essential Blocks repository | [Create a PAT](https://github.com/settings/tokens) with `repo` scope |

**To add the secret:**
1. Go to your repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `ESSENTIAL_BLOCKS_PAT`
4. Value: Your Personal Access Token
5. Click "Add secret"

### Repository Access

The workflow needs access to:
- `WPDevelopers/essential-blocks` (private repository)

---

## 🚀 Setup Instructions

### 1. Clone This Repository

```bash
git clone https://github.com/YOUR_USERNAME/php-test-Essential-blocks.git
cd php-test-Essential-blocks
```

### 2. Configure GitHub Secret

Add `ESSENTIAL_BLOCKS_PAT` secret as described in [Prerequisites](#prerequisites).

### 3. Customize PHP Versions (Optional)

Edit `.github/workflows/wp-phpcompat-full.yml`:

```yaml
strategy:
  matrix:
    php: [ 8.2 ]  # Change to test multiple versions: [7.2, 7.4, 8.0, 8.1, 8.2, 8.3, 8.4]
```

### 4. Push to GitHub

```bash
git add .
git commit -m "Setup PHP compatibility testing workflow"
git push origin main
```

---

## ▶️ Running the Workflow

### Via GitHub UI

1. Go to your repository on GitHub
2. Click **Actions** tab
3. Select **WP PHP Compatibility FULL** workflow
4. Click **Run workflow** button
5. Enter the Essential Blocks branch name (default: `master`)
6. Click **Run workflow**

### Via GitHub CLI

```bash
gh workflow run "WP PHP Compatibility FULL" -f branch=master
```

### Monitoring Progress

- Watch the workflow run in the Actions tab
- Each PHP version runs in parallel
- Total runtime: ~10-15 minutes per PHP version

---

## 📊 Understanding the Reports

### Downloading Reports

After the workflow completes:

1. Go to the workflow run page
2. Scroll to **Artifacts** section at the bottom
3. Download `essential-blocks-wp-compat-final`
4. Extract the ZIP file

### Report Files

```
essential-blocks-wp-compat-final/
├── combined.html              # Main report with all PHP versions
├── combined.pdf               # PDF version of the report
├── php-7.2-errors.html        # Individual error report (if errors found)
├── php-8.0-errors.html        # Individual error report (if errors found)
├── php-8.2-errors.html        # Individual error report (if errors found)
└── *.jpg                      # Screenshots from all PHP versions
```

### Report Structure

#### 1. Summary Table

Shows at-a-glance status for all PHP versions:

| Column | Description |
|--------|-------------|
| **PHP Version** | The PHP version tested |
| **Setup** | ✅ Success or ⚠️ Errors during WordPress/plugin setup |
| **Compatibility Status** | PASS, FAIL, or WARNING based on scan results |
| **Errors** | Number of PHPCompatibility errors found |
| **Warnings** | Number of PHPCompatibility warnings found |
| **Fatal Errors** | Number of pages with fatal errors |

#### 2. Detailed Results Per PHP Version

For each PHP version, the report shows:

**🔨 Plugin Build**
- Build process logs
- Submodule initialization
- pnpm install and build output
- Build verification status

**⚠️ Setup Status**
- WordPress installation logs
- Database creation
- Plugin activation
- Any setup errors encountered

**🔍 PHPCompatibility Check**
- Static code analysis results
- List of incompatible functions/features
- File locations and line numbers
- Severity (ERROR or WARNING)

**❌ Admin Pages Check**
- Fatal error detection on admin pages
- Pages tested:
  - Essential Blocks settings page
  - Post editor
  - Page editor
- Error details if found

**📝 WordPress Debug Log**
- PHP warnings and notices
- Deprecated function calls
- Database errors

**📸 Screenshots**
- Visual snapshots of admin pages
- Helps identify UI issues
- Shows actual page state during testing

#### 3. Individual Error Reports

Created only for PHP versions with errors:
- Focused view of just the errors
- Easier to share with developers
- Includes all error details and logs

### Status Badges

| Badge | Meaning |
|-------|---------|
| <span style="background: #d4edda; color: #155724; padding: 4px 12px; border-radius: 4px; font-weight: bold;">✅ PASS</span> | No issues found |
| <span style="background: #fff3cd; color: #856404; padding: 4px 12px; border-radius: 4px; font-weight: bold;">⚠️ WARNING</span> | Warnings found, but no errors |
| <span style="background: #f8d7da; color: #721c24; padding: 4px 12px; border-radius: 4px; font-weight: bold;">❌ FAIL</span> | Errors or fatal errors found |

---

## 🔧 Workflow Steps Explained

### Step 1: Checkout Essential Blocks

```yaml
- name: Checkout Essential Blocks plugin
  uses: actions/checkout@v4
  with:
    repository: WPDevelopers/essential-blocks
    ref: ${{ github.event.inputs.branch }}
    token: ${{ secrets.ESSENTIAL_BLOCKS_PAT }}
    path: essential-blocks
    submodules: recursive
```

**What it does:**
- Clones the Essential Blocks repository
- Checks out the specified branch
- Initializes git submodules (including `src/controls`)
- Uses PAT for authentication

### Step 2: Setup Node.js & pnpm

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '18'

- name: Install pnpm
  run: npm install -g pnpm
```

**What it does:**
- Installs Node.js version 18
- Installs pnpm package manager globally
- Required for building the plugin

### Step 3: Build Essential Blocks Plugin

```bash
# Initialize submodules
git submodule update --init --recursive

# Build src/controls first
cd src/controls
pnpm install
pnpm run build

# Build root project
cd ../..
pnpm install
pnpm run build
```

**What it does:**
- Initializes git submodules
- Builds the `src/controls` submodule first
- Builds the root project
- Logs all steps to `reports/{php-version}/build/`
- Continues even if build fails

**Why build before WordPress setup:**
- Plugin needs `.config/` directory and webpack config
- Submodules need git repository context
- Dependencies resolve correctly in original repo

### Step 4: Setup PHP & Tools

```yaml
- name: Setup PHP
  uses: shivammathur/setup-php@v2
  with:
    php-version: ${{ matrix.php }}
    tools: composer, phpcs, wp-cli
    extensions: mbstring, intl, curl, zip, xml, json
```

**What it does:**
- Installs the specified PHP version
- Installs Composer, PHP_CodeSniffer, WP-CLI
- Installs required PHP extensions

### Step 5: Install PHPCompatibilityWP

```bash
composer global require \
  "squizlabs/php_codesniffer:^3.0" \
  "dealerdirect/phpcodesniffer-composer-installer:*" \
  "phpcompatibility/phpcompatibility-wp:*"
```

**What it does:**
- Installs PHP_CodeSniffer 3.x
- Installs PHPCompatibilityWP coding standard
- Configures phpcs with correct installed paths

### Step 6: Setup MySQL & WordPress

```bash
# Start MySQL
sudo /etc/init.d/mysql start

# For PHP < 7.4, change MySQL authentication
if [[ "${{ matrix.php }}" < "7.4" ]]; then
  mysql -uroot -proot -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';"
fi

# Create database
mysql -e 'CREATE DATABASE wp;' -uroot -proot

# Download WordPress
wp core download --allow-root

# Create wp-config.php
wp config create --dbname=wp --dbuser=root --dbpass=root
```

**What it does:**
- Starts MySQL server
- Configures MySQL authentication for PHP < 7.4 compatibility
- Creates WordPress database
- Downloads latest WordPress
- Creates wp-config.php with debug settings

**MySQL Authentication Fix:**
- MySQL 8.0 uses `caching_sha2_password` by default
- PHP 7.2 doesn't support this authentication method
- Workflow automatically switches to `mysql_native_password` for PHP < 7.4

### Step 7: Install WordPress

```bash
wp core install \
  --url="http://127.0.0.1:8080" \
  --title="WP Test" \
  --admin_user="admin" \
  --admin_password="admin" \
  --admin_email="admin@example.com" \
  --skip-email \
  --allow-root
```

**What it does:**
- Installs WordPress
- Creates admin user (username: `admin`, password: `admin`)
- Configures site URL for local server

### Step 8: Copy & Activate Plugin

```bash
# Copy built plugin to WordPress
cp -R essential-blocks/* wp-content/plugins/essential-blocks/

# Update all plugins
wp plugin update --all --allow-root

# Activate Essential Blocks
wp plugin activate essential-blocks --allow-root
```

**What it does:**
- Copies the built plugin to WordPress plugins directory
- Updates all plugins to latest versions
- Activates Essential Blocks plugin

### Step 9: Start WordPress Server

```bash
wp server --host=127.0.0.1 --port=8080 --allow-root
```

**What it does:**
- Starts WordPress built-in PHP server
- Listens on `http://127.0.0.1:8080`
- Required for admin page checks and screenshots

### Step 10: Run PHPCompatibility Scan

```bash
phpcs \
  --standard=PHPCompatibilityWP \
  --runtime-set testVersion "8.2" \
  --extensions=php \
  --report=full \
  --report-file=reports/8.2/phpcompat/phpcompat-8.2.txt \
  wp-content/plugins/essential-blocks
```

**What it does:**
- Scans all PHP files in the plugin
- Checks for compatibility with the target PHP version
- Reports errors and warnings
- Saves results to text file

**Common issues detected:**
- Deprecated functions
- Removed functions
- New syntax not available in older PHP
- Changed function signatures

### Step 11: Check Admin Pages for Fatal Errors

```bash
# Pages checked:
# - http://127.0.0.1:8080/wp-admin/admin.php
# - http://127.0.0.1:8080/wp-admin/post-new.php?post_type=post
# - http://127.0.0.1:8080/wp-admin/post-new.php?post_type=page

# For each page:
curl -b cookies.txt "$url" -o page.html
grep -E "Fatal error|Parse error|Uncaught Error" page.html
```

**What it does:**
- Logs in to WordPress admin with curl
- Fetches admin pages
- Searches for PHP fatal errors in HTML output
- Logs results to `reports/{php-version}/fatal/`

**Error patterns detected:**
- `<b>Fatal error</b>:`
- `<b>Parse error</b>:`
- `Fatal error:`
- `Parse error:`
- `Uncaught Error:`
- `Uncaught Exception:`

### Step 12: Take Screenshots

```javascript
// Uses Puppeteer to:
// 1. Login to WordPress admin
// 2. Navigate to target pages
// 3. Take full-page screenshots
// 4. Save as JPEG images

const pages = [
  'Essential Blocks settings',
  'Post editor',
  'Page editor'
];
```

**What it does:**
- Launches headless Chrome browser
- Logs in with admin credentials
- Navigates to each admin page
- Captures 1920x1080 viewport screenshots
- Converts PNG to JPEG for smaller file size
- Saves to `reports/{php-version}/screens/`

**Why screenshots are useful:**
- Visual confirmation of page loading
- Detect UI/layout issues
- Verify plugin activation
- Debug authentication problems

### Step 13: Upload Artifacts

```yaml
- name: Upload per-PHP artifacts
  uses: actions/upload-artifact@v4
  with:
    name: perphp-reports-8.2
    path: reports/8.2
    retention-days: 7
```

**What it does:**
- Uploads all reports for this PHP version
- Stored as GitHub Actions artifact
- Available for 7 days
- Used by combine job

---

## 🛡️ Error Handling

### Graceful Failure

The workflow is designed to **continue even when steps fail**:

```bash
# Example: Continue on error
pnpm install 2>&1 | tee build.log || echo "Build failed" >> errors.log
```

**Benefits:**
- All PHP versions complete testing
- Partial results are still useful
- Errors are logged for debugging
- No workflow failures due to plugin issues

### Error Logging

All errors are logged to dedicated files:

```
reports/{php-version}/
├── setup/
│   ├── errors.log              # Summary of all setup errors
│   ├── mysql-config.log        # MySQL authentication
│   ├── db-create.log           # Database creation
│   ├── wp-download.log         # WordPress download
│   ├── wp-config.log           # wp-config.php creation
│   ├── wp-install.log          # WordPress installation
│   ├── plugin-copy.log         # Plugin copy
│   ├── plugin-update.log       # Plugin update
│   ├── plugin-activate.log     # Plugin activation
│   └── app-password.log        # Application password
├── build/
│   ├── build.log               # Build process summary
│   ├── submodule.log           # Git submodule init
│   ├── controls-install.log    # src/controls pnpm install
│   ├── controls-build.log      # src/controls build
│   ├── root-install.log        # Root pnpm install
│   └── root-build.log          # Root build
├── phpcompat/
│   └── phpcompat-8.2.txt       # PHPCompatibility scan results
├── fatal/
│   ├── fatal-8.2.log           # Fatal error check results
│   ├── admin.html              # Admin page HTML
│   ├── editor_post.html        # Post editor HTML
│   └── editor_page.html        # Page editor HTML
├── wpdebug/
│   └── debug-8.2.log           # WordPress debug.log
└── screens/
    ├── 8.2_admin.jpg           # Admin page screenshot
    ├── 8.2_editor_post.jpg     # Post editor screenshot
    └── 8.2_editor_page.jpg     # Page editor screenshot
```

### Error Reporting

Errors are reported in multiple ways:

1. **Setup Errors Section** - Shows which setup steps failed
2. **Build Logs** - Expandable sections with full build output
3. **Individual Error Reports** - Separate HTML files per PHP version
4. **Summary Table** - At-a-glance status with ⚠️ warnings

---

## 🔍 Troubleshooting

### Common Issues

#### 1. Workflow Fails to Start

**Problem:** Workflow doesn't run when triggered

**Solutions:**
- ✅ Check that `ESSENTIAL_BLOCKS_PAT` secret is configured
- ✅ Verify PAT has `repo` scope
- ✅ Ensure PAT hasn't expired
- ✅ Check workflow file syntax with `yamllint`

#### 2. Plugin Build Fails

**Problem:** Build step shows errors in logs

**Common causes:**
- Missing dependencies in `package.json`
- Incompatible Node.js version
- Submodule not initialized
- Missing `.config/` directory

**Solutions:**
- ✅ Check `build.log` for specific errors
- ✅ Verify submodules are initialized
- ✅ Ensure Node.js 18 is used
- ✅ Check that build works locally

#### 3. MySQL Authentication Error (PHP 7.2)

**Problem:** `The server requested authentication method unknown to the client`

**Solution:**
- ✅ Workflow automatically handles this for PHP < 7.4
- ✅ Check `mysql-config.log` for authentication change
- ✅ Verify MySQL started successfully

#### 4. Screenshots Show Login Page

**Problem:** Screenshots show wp-login.php instead of admin pages

**Common causes:**
- Authentication failed
- Session cookies not working
- WordPress not fully installed

**Solutions:**
- ✅ Check `wp-install.log` for installation errors
- ✅ Verify admin user was created
- ✅ Check Puppeteer login logs in workflow output

#### 5. No Artifacts Generated

**Problem:** No artifacts available after workflow completes

**Solutions:**
- ✅ Check if workflow completed successfully
- ✅ Verify all jobs finished (not cancelled)
- ✅ Check artifact retention period (7 days for per-PHP, 30 days for final)

#### 6. PDF Generation Fails

**Problem:** `combined.pdf` not created

**Solutions:**
- ✅ Check wkhtmltopdf installation
- ✅ Verify HTML file is valid
- ✅ Check for file access permissions
- ✅ HTML report is still available even if PDF fails

---

## 🔬 Technical Details

### PHP Versions Tested

| Version | Status | Notes |
|---------|--------|-------|
| 7.4 | ✅ Supported | Minimum supported version |
| 8.0 | ✅ Supported | |
| 8.1 | ✅ Supported | |
| 8.2 | ✅ Supported | |
| 8.3 | ✅ Supported | |
| 8.4 | ✅ Supported | Latest version |

### Tools & Dependencies

| Tool | Version | Purpose |
|------|---------|---------|
| Node.js | 18 | JavaScript runtime for building plugin |
| pnpm | Latest | Fast, disk-efficient package manager |
| PHP | Matrix | Target PHP version for testing |
| Composer | Latest | PHP dependency manager |
| WP-CLI | Latest | WordPress command-line interface |
| PHP_CodeSniffer | 3.x | Code analysis framework |
| PHPCompatibilityWP | Latest | WordPress-specific PHP compatibility rules |
| Puppeteer | Latest | Headless browser for screenshots |
| wkhtmltopdf | Latest | HTML to PDF converter |
| MySQL | 8.0 | Database server (pre-installed on runner) |

### GitHub Actions Runner

- **OS:** Ubuntu Latest
- **Architecture:** x64
- **Pre-installed:** MySQL 8.0, Git, curl, wget
- **Resources:** 2-core CPU, 7 GB RAM, 14 GB SSD

### Workflow Execution Time

| Step | Approximate Time |
|------|------------------|
| Checkout & Setup | 1-2 minutes |
| Plugin Build | 2-3 minutes |
| WordPress Setup | 2-3 minutes |
| PHPCompatibility Scan | 1-2 minutes |
| Admin Page Checks | 1 minute |
| Screenshots | 2-3 minutes |
| **Total per PHP version** | **10-15 minutes** |
| Combine Reports | 1-2 minutes |
| **Total (1 PHP version)** | **11-17 minutes** |
| **Total (7 PHP versions)** | **11-17 minutes** (parallel) |

### File Sizes

| File | Approximate Size |
|------|------------------|
| Per-PHP artifact | 5-10 MB |
| Screenshot (JPEG) | 200-500 KB |
| combined.html | 500 KB - 2 MB |
| combined.pdf | 1-5 MB |
| Individual error report | 100-500 KB |

### Network Usage

- WordPress download: ~20 MB
- pnpm dependencies: ~100-200 MB
- Puppeteer (Chromium): ~150 MB
- Total per run: ~300-400 MB

---

## 📝 Configuration Options

### Customizing PHP Versions

Edit `.github/workflows/wp-phpcompat-full.yml`:

```yaml
strategy:
  matrix:
    # Test single version
    php: [ 8.2 ]

    # Test multiple versions
    php: [ 7.4, 8.0, 8.2 ]

    # Test all supported versions
    php: [ 7.4, 8.0, 8.1, 8.2, 8.3, 8.4 ]
```

### Customizing Branch

Change the default branch in workflow file:

```yaml
inputs:
  branch:
    description: 'Essential Blocks branch to test'
    required: true
    default: 'master'  # Change this
```

Or specify when running:
```bash
gh workflow run "WP PHP Compatibility FULL" -f branch=develop
```

### Customizing Pages to Check

Edit the `TARGETS` array in the workflow:

```bash
TARGETS=(
  "http://127.0.0.1:8080/wp-admin/admin.php admin"
  "http://127.0.0.1:8080/wp-admin/post-new.php?post_type=post editor_post"
  "http://127.0.0.1:8080/wp-admin/post-new.php?post_type=page editor_page"
  # Add more pages:
  "http://127.0.0.1:8080/wp-admin/plugins.php plugins"
)
```

### Customizing Artifact Retention

```yaml
- name: Upload per-PHP artifacts
  uses: actions/upload-artifact@v4
  with:
    retention-days: 7  # Change this (1-90 days)

- name: Upload FINAL combined artifact
  uses: actions/upload-artifact@v4
  with:
    retention-days: 30  # Change this (1-90 days)
```

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the workflow
5. Submit a pull request

---

## 📄 License

This project is licensed under the MIT License.

---

## 🙏 Acknowledgments

- **Essential Blocks Team** - For the amazing WordPress plugin
- **PHPCompatibility** - For the excellent compatibility checking tool
- **WordPress** - For the powerful CMS platform
- **GitHub Actions** - For the CI/CD infrastructure

---

## 📞 Support

For issues or questions:

1. Check the [Troubleshooting](#troubleshooting) section
2. Review workflow logs in GitHub Actions
3. Open an issue in this repository

---

**Made with ❤️ for WordPress Security & QA**