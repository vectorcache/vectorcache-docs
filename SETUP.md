# Vectorcache Documentation Setup Guide

This guide will help you publish the Vectorcache documentation to GitHub Pages.

## Step 1: Create GitHub Repository

1. Go to [GitHub](https://github.com) and log in
2. Click the **+** icon → **New repository**
3. Repository details:
   - **Name**: `vectorcache-docs`
   - **Description**: Official documentation for Vectorcache
   - **Visibility**: Public (required for free GitHub Pages)
   - **DO NOT** initialize with README, .gitignore, or license (we already have these)
4. Click **Create repository**

## Step 2: Update Configuration

Before pushing, update these placeholders in the files:

### In `mkdocs.yml` (line 7-8):
```yaml
repo_name: vectorcache/vectorcache-docs
repo_url: https://github.com/YOUR_USERNAME/vectorcache-docs
```

Replace `YOUR_USERNAME` with your GitHub username.

### In `mkdocs.yml` (line 75-76):
```yaml
- icon: fontawesome/brands/github
  link: https://github.com/YOUR_USERNAME/vectorcache-docs
```

Replace `YOUR_USERNAME` with your GitHub username.

## Step 3: Initialize and Push

Run these commands from the `vectorcache-docs` directory:

```bash
# Navigate to the docs directory
cd /Users/rionangeles/Documents/projects/vectorcache-workspace/vectorcache-docs

# Add all files
git add .

# Commit
git commit -m "Initial documentation setup"

# Add remote (replace YOUR_USERNAME)
git remote add origin https://github.com/YOUR_USERNAME/vectorcache-docs.git

# Push to GitHub
git branch -M main
git push -u origin main
```

## Step 4: Enable GitHub Pages

1. Go to your repository on GitHub
2. Click **Settings** (top menu)
3. Scroll to **Pages** (left sidebar under "Code and automation")
4. Under **Build and deployment**:
   - **Source**: Deploy from a branch
   - **Branch**: `gh-pages` → `/` (root)
   - Click **Save**

**Note**: The `gh-pages` branch will be created automatically by the GitHub Actions workflow after the first push.

## Step 5: Wait for Deployment

1. Go to the **Actions** tab in your repository
2. You should see a workflow running: "Deploy MkDocs to GitHub Pages"
3. Wait for it to complete (usually 1-2 minutes)
4. Once complete, the `gh-pages` branch will be created

## Step 6: Configure GitHub Pages (Again)

After the first deployment:

1. Go back to **Settings** → **Pages**
2. Verify the source is set to `gh-pages` branch
3. Your site URL will be shown: `https://YOUR_USERNAME.github.io/vectorcache-docs/`

## Step 7: Custom Domain (Optional)

To use `docs.vectorcache.com`:

### DNS Configuration:
1. Add a CNAME record in your DNS:
   - **Name**: `docs`
   - **Value**: `YOUR_USERNAME.github.io`

### GitHub Configuration:
1. Go to **Settings** → **Pages**
2. Under **Custom domain**, enter: `docs.vectorcache.com`
3. Click **Save**
4. Wait for DNS check to complete
5. Enable **Enforce HTTPS** (after DNS propagates)

### Update mkdocs.yml:
```yaml
site_url: https://docs.vectorcache.com
```

Commit and push the change.

## Step 8: Verify

Visit your documentation site:
- GitHub Pages: `https://YOUR_USERNAME.github.io/vectorcache-docs/`
- Custom domain (if configured): `https://docs.vectorcache.com`

## Local Development

Test locally before pushing:

```bash
# Install dependencies
pip install -r requirements.txt

# Run local server
mkdocs serve

# Open http://localhost:8000
```

## Updating Documentation

To update the docs:

1. Make changes to files in `docs/` directory
2. Test locally: `mkdocs serve`
3. Commit and push:
   ```bash
   git add .
   git commit -m "Update documentation"
   git push
   ```
4. GitHub Actions will automatically rebuild and deploy

## Troubleshooting

### Actions Workflow Fails

If the GitHub Actions workflow fails:

1. Go to **Actions** tab
2. Click on the failed workflow
3. Check the error message
4. Common fixes:
   - Ensure `requirements.txt` has correct dependencies
   - Verify all markdown files are valid
   - Check `mkdocs.yml` for syntax errors

### GitHub Pages Not Showing

1. Verify **Settings** → **Pages** is configured
2. Check that `gh-pages` branch exists
3. Look for deployment in **Actions** tab
4. Clear browser cache and try again

### Custom Domain Not Working

1. Verify DNS CNAME record: `dig docs.vectorcache.com`
2. Wait for DNS propagation (can take up to 48 hours)
3. Check **Settings** → **Pages** for DNS verification status
4. Ensure CNAME file exists in `gh-pages` branch (created automatically)

## Support

Need help?

- Check the [MkDocs Material documentation](https://squidfunk.github.io/mkdocs-material/)
- Review [GitHub Pages documentation](https://docs.github.com/en/pages)
- Open an issue in the repository

---

**Ready to publish?** Follow the steps above to get your documentation live! 🚀
