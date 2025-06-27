---
title: "Personal Website with Hugo & GitHub: Part 2 – GitHub Pages & Custom Domain"
date: 2025-06-27
---

This is Part 2 of my series on building a personal website with Hugo and GitHub. In this post, you'll learn how to take your Hugo site live with GitHub Pages, set up a custom domain (using NameCheap as an example), and configure Hugo for production. If you haven't completed the initial setup, make sure to start with [← Part 1: Project Setup and Theme](/posts/hugo-github-setup/).

---

## Bringing It together: GitHub Pages, Custom Domain, and Hugo

### Set up host records for your custom domain
I use NameCheap, but steps are similar for other providers.

1. Sign in and go to your Dashboard.
2. Go to the domain list and click on **Manage** for your domain (e.g., tristan.run).
3. Go to **Advanced DNS** and change the **CNAME Record** to your username.github.io. (e.g., `imtris.github.io.` — note the trailing dot).
4. Add the following **A records** for `@`:
   - 185.199.108.153
   - 185.199.109.153
   - 185.199.110.153
   - 185.199.111.153

> **Note:** The CNAME record should have a trailing dot: `imtris.github.io.` is correct, `imtris.github.io` is not.

### Set up GitHub Pages
1. Go to your repository and click on **Settings**.
2. Go to the **Pages** section (currently under "Code and automation").
3. Under **Build and deployment**, select **Source: GitHub Actions**.
4. Add your website in the custom domain field (e.g., `www.tristan.run`), press save, and wait for the DNS check to finish.
5. After the check, enable the **Enforce HTTPS** box.

### Update Hugo image cache settings
In your `hugo.toml` file, add:
```toml
[caches]
  [caches.images]
    dir = ':cacheDir/images'
```

### Add a GitHub Actions workflow
Create `.github/workflows/hugo.yaml` with:
```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches:
      - master

  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.147.2
      HUGO_ENVIRONMENT: production
      TZ: America/Los_Angeles
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Cache Restore
        id: cache-restore
        uses: actions/cache/restore@v4
        with:
          path: |
            ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-
      - name: Configure Git
        run: git config core.quotepath false
      - name: Build with Hugo
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
      - name: Cache Save
        id: cache-save
        uses: actions/cache/save@v4
        with:
          path: |
            ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Commit the changes and check the **Actions** menu in GitHub. It should show your newly added workflow.

---

## References
- [NameCheap: Link Domain to GitHub Pages](https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages/)
- [GitHub Pages Quickstart](https://docs.github.com/en/pages/quickstart)
- [42 Steps from Zero to an Automated GitHub Website Built with Hugo](https://selfscrum.medium.com/42-steps-from-zero-to-an-automated-github-website-built-with-hugo-2fa001827db1)
- [Host on GitHub Pages (Hugo Docs)](https://gohugo.io/host-and-deploy/host-on-github-pages/)
