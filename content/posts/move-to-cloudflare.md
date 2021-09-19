---
title: "Bye Bye Google Blogger🖐, Hello Cloudflare Pages😘"
date: 2021-09-04T11:50:12+10:00
draft: false
---

I had enough of Google Blogger! It has terrible editing UI for and I constantly receiving SPAM comments for my posts😤. Time to move my blog off to somewhere better! I was thinking to run a Wordpress server. Though I think it's way cooler to run my blog simplely without worrying about backend infrastructure.

After look around, I ended up using Cloudflare Pages to publish my blog. This allows me to write my posts with `Markdown` and then generate static html pages using [Hugo](gohugo.io). The pages are then published using GitHub flow which in turn triggers a new build in Cloudflare Page. It's totally version controlled and I don't need to worry about maintaining any infrastructure. 

So here's how I set everything up.
1. Install `Hugo`.
2. Creat a new GitHub repo.
3. Clone the repo to local.
4. Create a new site using Hugo
```bash
hugo new site sitename
```
5. Download a theme to `myblog/sitename/themes` folder. I choose [hugo-geekblog](https://themes.gohugo.io/themes/hugo-geekblog/). You can choose whichever you like. Though be aware they all work slightly different.
6. Create a Cloudflare Page
7. Configure the Cloudflare Page build settings as below.
```
Production branch: main
Preview branches: All non-production branches
Build command: hugo
Build output directory: /public
Root directory: /
```
8. I have to specify Hugo Version as `hugo-geekblog` theme only supports hugo version 0.65+.
```
HUGO_VERSION 0.87.0
```
9. Create a post by running:
```bash
hugo new post/postname.md
```
10. This creates a md file under `content/posts/postname.md`.
11. Once complete the post commit and push the changes to GitHub repo.
12. To redirect traffic from `tomking.xyz` to my new site, simply setup a custom domain in `Pages`. 

To migrate all my existing contents from Blogger, I use [blog2md](https://github.com/palaniraja/blog2md). From what I can see most pictures were retained!