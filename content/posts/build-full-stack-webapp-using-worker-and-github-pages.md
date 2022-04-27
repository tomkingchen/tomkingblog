---
title: "Build Full Stack Webapp Using Cloudflare Worker and Github Pages"
date: 2022-04-20T08:07:31+10:00
draft: false
---

I have been following the COVID situation in Shanghai lately. Intentionally or not, it is surprisingly hard to find a daily chart to reflect the proper case numbers overthere. To solve this problem, I decided to build a webapp just does exactly that: a simple chart the displays Shanghai daily COVID cases that includes both symptomatic and asymptomatic cases.

The overall design includes a backend API that provides daily numbers in JSON format, and a front-end page that presents the data in the form of an area chart.

![Overall Design](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-04-2022/shanghaicovid.drawio.png)

To start, I created the backend API using Cloudflare Worker. Cloudflare provided a [very detailed document](https://developers.cloudflare.com/pages/tutorials/build-an-api-with-workers/) for this exact purpose. I simply follow the first half of this document to build up my API. There are a few small issues you might run into if you try to follow the exact steps described in the document. Let me highlight them for you here.

### Install wrangler
The official document indicates to use `npm i @cloudflare/wrangler -g`, which can cause permission issue in Linux. Instead, use `yarn global add @cloudflare/wrangler` to install the package.
## Setup GitHub Pages for Front-end
The Cloudflare document provides steps to deploy the Front-end with [Cloudflare Pages](https://developers.cloudflare.com/pages) service. Which in my case is not possible, as Page in my account has already configured for this very blog. A great alternative is GitHub Pages (Yes, I know. Another "Pages", people need to be more creative in naming their products). Here are the steps I took to set the repo up.

### Create the project in GitHub Repo
Nothing fancy here, just create a `public` repo as per usual. Initial the repo with `npx create-react-app .` and then start coding. 

### Configure GitHub Pages
Install `gh-pages` as part of the dev packages with `yarn add -d gh-pages`. 

`gh-pages` will automatically create the `gh-branch` which hosts the post-build files that ready to be published through GitHub Pages.

In `package.json`, add following lines:
```diff
{
  "name": "shanghai-rona-front",
+ "homepage": "https://shanghaicovid.xyz",
  "version": "0.1.0",
...
"scripts": {
+   "predeploy": "yarn build",
+   "deploy": "gh-pages -d build",
    "start": "react-scripts start",
```

### Setup Custom Domain
By default GitHub Pages use the `foo.github.io/project-name` as the public domain name. To change this, first create a CNAME record for `www.shanghaicovid.xyz` in Cloudflare and point it to `tomkingchen.github.io`. Next, create a txt file named as `CNAME` and save it in `./public` folder along with other static files in the folder.

In the file, simply put in the custom domain name.
```
www.shanghaicovid.xyz
```
The file will be picked up by `gh-pages` during deploy process.

### Publish the site
To publish the React site, we run `yarn deploy` which will kick off the local build and update `gh-pages` branch. Once that's done, run `git push` to push the local branch to GitHub. GitHub will then configure Pages based on contents in the `gh-pages` branch.

### Setup CICD using GitHub Actions
One of the advantages to use GitHub Pages is we can automate the CICD build using GitHub Actions. To do so, create `.github/workflows/` folder. Create a `deploy.yml` file in it. Below are the steps I use in my pipeline. The only tricky bit is how to push changes to `gh-pages` branch within the pipeline. Fortuantely I found this `ghaction-github-pages` action does exactly that.
```yaml
name: Deploy

on: [push, pull_request]

jobs:
  deploy:
    name: Deploy GitHub Page
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Checkout Repo
        uses: actions/checkout@master

      - name: Setup Node.js 16.14.0
        uses: actions/setup-node@master
        with:
          node-version: 16.14.0

      - name: Install Dependencies
        run: yarn --frozen-lockfile

      - run: yarn build
      - name: Deploy
        uses: crazy-max/ghaction-github-pages@v1
        with:
          target_branch: gh-pages
          build_dir: build
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Source Code
You can find the front-end source code from [here](https://github.com/tomkingchen/shanghai-rona-front)