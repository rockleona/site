---
title: GitHub Pages
---

In this tutorial, we use [GitHub Actions](https://docs.github.com/en/actions) to deploy GitHub Pages. It works in both public and private repositories. Skip to the [One-command deployment](#One-command-deployment) section if you prefer not to upload your source folder to GitHub.

1. Create a repo named <b>*username*.github.io</b>, where username is your username on GitHub. If you have already uploaded to another repo, rename the repo instead.
2. Push the files of your Hexo folder to the default branch of your repository. The default branch is usually **main**, older repositories may use **master** branch.
  - To push `main` branch to GitHub:

    ```
    $ git push -u origin main
    ```
  - The `public/` folder is not (and should not be) uploaded by default, make sure the `.gitignore` file contains `public/` line. The folder structure should be roughly similar to [this repo](https://github.com/hexojs/hexo-starter).

3. Check what version of Node.js you are using on your local machine with `node --version`. Make a note of the major version (e.g., `v20.y.z`)
4. In your GitHub repo's setting, navigate to **Settings** > **Pages** > **Source**. Change the source to **GitHub Actions** and save.
5. Create `.github/workflows/pages.yml` in your repo with the following contents (substituting `20` to the major version of Node.js that you noted in previous step):

```yml .github/workflows/pages.yml
name: Pages

on:
  push:
    branches:
      - main  # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: '20'
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

6. Once the deployment is finished, check the webpage at *username*.github.io.

Note - if you specify a custom domain name with a `CNAME`, you need to add the `CNAME` file to the `source/` folder. [More info](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

## Project page

If you prefer to have a project page on GitHub:

1. Navigate to your repo on GitHub. Go to the **Settings** tab. Change the **Repository name** so your blog is available at <b>username.github.io/*repository*</b>,  **repository** can be any name, like *blog* or *hexo*.
2. Edit your **_config.yml**, change the `url:` value to <b>https://*username*.github.io/*repository*</b>.
3. In your GitHub repo's setting, navigate to **Settings** > **Pages** > **Source**. Change the source to **GitHub Actions** and save.
4. Commit and push to the default branch.
4. Once the deployment is finished, check the webpage at *username*.github.io/*repository*.

## One-command deployment

The following instruction is adapted from [one-command deployment](/docs/one-command-deployment) page.

1. Install [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git).
2. Add the following configurations to **_config.yml**, (remove existing lines if any).

  ``` yml
  deploy:
    type: git
    repo: https://github.com/<username>/<project>
    # example, https://github.com/hexojs/hexojs.github.io
    branch: gh-pages
  ```

3. Run `hexo clean && hexo deploy`.
4. Check the webpage at *username*.github.io.

## Useful links

- [GitHub Pages](https://docs.github.com/en/pages)
- [Publishing with a custom GitHub Actions workflow](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow)
- [actions/deploy-github-pages-site](https://github.com/marketplace/actions/deploy-github-pages-site)
