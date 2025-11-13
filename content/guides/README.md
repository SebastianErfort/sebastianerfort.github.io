---
title: Website with Hugo and Blowfish theme
date: 2024-07-07
---

1. follow instructions in [Blowfish repo/on website][blowfish] to set up Hugo and theme
2. configure: files `config/_default/*.toml`
3. edit landing page `content/_index.md`

## CI/CD

I tried the suggested [GitHub workflow from the Blowfish docs](https://blowfish.page/docs/hosting-deployment/#github-pages), but I found the instructions a bit incomplete and possibly outdated. Checkout, setting up Hugo and building all worked fine, but deployment failed. The action used in this step noticed the `gh-pages` branch missing and created it, but deployment failed. I tried expanding the `permissions` for the `GITHUB_TOKEN` with `write: true`, created the missing branch in the repo (empty orphan) and tried deployment from the branch `gh-pages` as stated in the Blowfish docs.

In the end I went back to the workflow I had been using for my [previous website with the Hugo Clarity theme](https://github.com/SebastianErfort/homepage-hugo-clarity). I tidied that one, removing some unnecessary dependencies and setting up Hugo through an action. For now I opted for the latest version of Ubuntu and Hugo, might pin versions some time for stability.

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between
# the run in-progress and latest queued. However, do NOT cancel
# in-progress runs as we want to allow these production deployments
# to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "latest"

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3

      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v2
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
    
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v2
```


## References

[Blowfish] theme for Hugo.

[blowfish]: <https://blowfish.page/>
