# 📄 GitHub Pages Setup for PHP Compatibility Reports

This guide explains how to host your PHP compatibility reports (combined.html only) on GitHub Pages, making them accessible via a public URL without downloading artifacts.

---

## 🎯 What This Does

After each workflow run, **only the combined.html report** will be automatically deployed to GitHub Pages at:

```
https://<your-username>.github.io/<repo-name>/
```

**What's Deployed:**
- ✅ `combined.html` only (renamed to `index.html`)
- ❌ PDF files (not deployed - available in artifacts)
- ❌ Individual error reports (not deployed - available in artifacts)
- ❌ Screenshots (not deployed - available in artifacts)

**Features:**
- ✅ **Always Latest** - Each deployment replaces the previous one
- ✅ **Single URL** - Always the same URL for the latest report
- ✅ **No Downloads** - View reports directly in browser
- ✅ **Easy Sharing** - Share URLs with team members
- ✅ **Lightweight** - Only HTML, no large files
- ✅ **Clean History** - Old reports are replaced (not kept)

---

## 🔧 Setup Instructions

### **Step 1: Enable GitHub Pages**

1. Go to your repository on GitHub
2. Click **Settings** → **Pages** (in left sidebar)
3. Under **Source**, select:
   - **Source:** `Deploy from a branch`
   - **Branch:** `gh-pages`
   - **Folder:** `/ (root)`
4. Click **Save**

### **Step 2: Run the Workflow**

1. Go to **Actions** tab
2. Select **WP PHP Compatibility FULL** workflow
3. Click **Run workflow**
4. Select branches and click **Run workflow**

### **Step 3: Wait for Deployment**

- The workflow will complete in ~14 minutes
- GitHub Pages deployment takes an additional 1-2 minutes
- You'll see a new deployment in **Settings → Pages**

### **Step 4: Access Your Report**

Your report will be available at:

```
https://<your-username>.github.io/<repo-name>/
```

**Example:**
```
https://mdismail.github.io/php-test-Essential-blocks/
```

---

## 📂 URL Structure

### **Latest Report (Always Same URL)**
```
https://<your-username>.github.io/<repo-name>/
```
- **Always shows the most recent report**
- Each new workflow run replaces the previous report
- Bookmark this URL for quick access
- Example: `https://mdismail-cse.github.io/php-test-Essential-blocks/`

### **What About Other Files?**

**PDF, Screenshots, Individual Error Reports:**
- ❌ Not deployed to GitHub Pages
- ✅ Available in GitHub Actions artifacts
- Download from: Actions → Workflow Run → Artifacts → `essential-blocks-wp-compat-final`

### **What About Report History?**

- ❌ Old reports are **NOT kept** on GitHub Pages
- ✅ Each deployment replaces the previous one
- ✅ Old reports are still available in GitHub Actions artifacts (30 days retention)

---

## 🔍 How It Works

### **Deployment Process**

1. **Workflow completes** → Generates `final/` directory with reports
2. **Deploy to GitHub Pages** → Pushes to `gh-pages` branch
   - Creates `reports/<run-number>/` directory
   - Copies all files (HTML, PDF, screenshots)
3. **Create redirect** → Updates root `index.html` to redirect to latest report
4. **GitHub Pages builds** → Makes files accessible via HTTPS

### **File Structure on gh-pages Branch**

```
gh-pages/
├── .nojekyll                            # Prevents Jekyll processing
└── index.html                           # Latest combined.html report
```

**Note:**
- Only `combined.html` is deployed (renamed to `index.html`)
- Each deployment completely replaces the previous one
- No subdirectories, no old reports kept

---

## 🛠️ Troubleshooting

### **Issue: 404 Page Not Found**

**Cause:** GitHub Pages not enabled or wrong branch selected

**Solution:**
1. Go to **Settings → Pages**
2. Ensure **Source** is set to `gh-pages` branch
3. Wait 1-2 minutes for deployment

### **Issue: Images Not Loading**

**Cause:** Screenshots are not deployed to GitHub Pages

**Solution:**
- Screenshots are only available in artifacts
- Download the full artifact to view screenshots
- Only the HTML report is hosted on GitHub Pages

### **Issue: Need to Access Old Reports**

**Cause:** Old reports are replaced with each deployment

**Solution:**
- Download artifacts from previous workflow runs
- Artifacts are kept for 30 days
- Go to Actions → Select workflow run → Download artifact

### **Issue: PDF Not Generated**

**Cause:** `wkhtmltopdf` failed

**Solution:**
- HTML report is still available
- PDF generation is optional
- Check workflow logs for errors

---

## 🔒 Security Considerations

### **Public vs Private Repositories**

- **Public Repo:** Reports are publicly accessible
- **Private Repo:** Reports require GitHub authentication

### **Making Reports Private**

If you need to restrict access:

1. **Option 1:** Keep repository private
   - Only collaborators can access reports
   - Requires GitHub login

2. **Option 2:** Use password protection
   - Add authentication to HTML reports
   - More complex setup

3. **Option 3:** Use artifacts only
   - Remove GitHub Pages deployment
   - Download reports from Actions artifacts

---

## 📊 Benefits Over Artifacts

| Feature | Artifacts | GitHub Pages |
|---------|-----------|--------------|
| **Access** | Download required | Direct browser access |
| **Sharing** | Send files | Send URL |
| **History** | 30 days retention | Permanent |
| **Mobile** | Difficult | Easy |
| **Search Engines** | No | Yes (if public) |

---

## 🎨 Customization

### **Change Report Path**

Edit line 828 in workflow:
```yaml
destination_dir: reports/${{ github.run_number }}
```

Change to:
```yaml
destination_dir: my-custom-path/${{ github.run_number }}
```

### **Add Custom Domain**

1. Add `CNAME` file to `final/` directory
2. Configure custom domain in **Settings → Pages**

---

## 📝 Next Steps

1. ✅ Enable GitHub Pages (see Step 1)
2. ✅ Run workflow
3. ✅ Access your report
4. ✅ Share URL with team
5. ✅ Bookmark latest report URL

---

**Questions?** Check the workflow logs in the Actions tab for deployment details.

