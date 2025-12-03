# 📄 GitHub Pages Setup for PHP Compatibility Reports

This guide explains how to host your PHP compatibility reports on GitHub Pages, making them accessible via a public URL without downloading artifacts.

---

## 🎯 What This Does

After each workflow run, the combined HTML report (with screenshots and all PHP version results) will be automatically deployed to GitHub Pages at:

```
https://<your-username>.github.io/<repo-name>/
```

**Features:**
- ✅ **Permanent URLs** - Each report gets a unique URL based on run number
- ✅ **Latest Report** - Root URL always redirects to the most recent report
- ✅ **History** - All previous reports remain accessible
- ✅ **No Downloads** - View reports directly in browser
- ✅ **Easy Sharing** - Share URLs with team members

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

### **Latest Report (Auto-Redirect)**
```
https://<your-username>.github.io/<repo-name>/
```
- Always redirects to the most recent report
- Bookmark this URL for quick access

### **Specific Report by Run Number**
```
https://<your-username>.github.io/<repo-name>/reports/<run-number>/combined.html
```
- Example: `https://mdismail.github.io/php-test-Essential-blocks/reports/42/combined.html`
- Permanent URL for a specific test run
- Useful for tracking issues over time

### **Individual Error Reports**
```
https://<your-username>.github.io/<repo-name>/reports/<run-number>/php-8.2-errors.html
```
- Only created if that PHP version had errors
- Direct link to share with developers

### **Screenshots**
```
https://<your-username>.github.io/<repo-name>/reports/<run-number>/8.2_admin.jpg
```
- All screenshots are accessible
- Embedded in HTML reports

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
├── index.html                           # Redirects to latest report
├── reports/
│   ├── 1/                               # Run #1
│   │   ├── combined.html
│   │   ├── combined.pdf
│   │   ├── php-8.2-errors.html
│   │   └── *.jpg
│   ├── 2/                               # Run #2
│   │   └── ...
│   └── 42/                              # Run #42 (latest)
│       ├── combined.html
│       ├── combined.pdf
│       └── ...
```

---

## 🛠️ Troubleshooting

### **Issue: 404 Page Not Found**

**Cause:** GitHub Pages not enabled or wrong branch selected

**Solution:**
1. Go to **Settings → Pages**
2. Ensure **Source** is set to `gh-pages` branch
3. Wait 1-2 minutes for deployment

### **Issue: Images Not Loading**

**Cause:** Relative paths in HTML

**Solution:** 
- Images are copied to the same directory as HTML
- Should work automatically
- Check browser console for errors

### **Issue: Old Reports Not Accessible**

**Cause:** `keep_files: true` not set

**Solution:**
- Already configured in workflow
- Each deployment preserves previous reports
- Check `gh-pages` branch to verify files exist

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

