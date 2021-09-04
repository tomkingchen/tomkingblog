---
title: "Publish my blog using Cloudflare Pages"
date: 2021-09-04T11:50:12+10:00
draft: false
---

Google's `Blogger` provide terrible edit experience and is subject to tons of SPAM comments. So I have decided to move my blog off to a different place. I was thinking to run a Wordpress server myself. Setting up the server and database will be a fun exercise. Though I am not a huge fan of PHP. And I had enough of patching servers🤮. Thesedays I work a lot with Cloudflare. So I think it will be helpful to have bit play with Cloudflare Pages.

So here's how I set everything up.
1. Install [Hugo](gohugo.io).
2. Creat a new GitHub repo https://github.com/tomkingchen/myblog.
3. Clone the repo to local.
4. Create a new site using Hugo
```bash
hugo new site sitename
```
5. Download a theme to `myblog/sitename/themes` folder. I choose [hugo-geekblog](https://themes.gohugo.io/themes/hugo-geekblog/). You can choose whichever you like. Though be aware they all work slightly different.
6. Create a Cloudflare Page
7. Configure the Cloudflare Page build settings as below
```
Build command: hugo
Build output directory: /public
Root directory: /sitename
```
8. I have to specify Hugo Version as `hugo-geekblog` theme only supports hugo version 0.65+

## Code block
This is some testing Python code.
```python
a = 100
b = "200"
def afunction(a,b) :
c = a + b
print("The result is: $c")

```
This is some testing Javascript code.
```javascript
const a = 100
const b = "200"
afunction (a,b) => {
  c = a + b
  console.log("The result is: "+ c)
}
```

## A Image

This is a ![image](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/analytic.png)

