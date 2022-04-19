---
title: "Build Full Stack Webapp Using Cloudflare Worker and Github Pages"
date: 2022-04-20T08:07:31+10:00
draft: true
---

I have been following the COVID situation in Shanghai lately. Intentionally or not, it is surprisingly hard to find a daily chart to reflect the proper case numbers overthere. To solve this problem, I decided to build a webapp just does exactly that: a simple chart the displays Shanghai daily COVID cases that includes both symptomatic and asymptomatic cases.

The overall design includes a backend API that provides daily numbers in JSON format, and a frontend page that presents the data in the form of an area chart.

To start, I created the backend API using Cloudflare Worker. Cloudflare provided a (very detailed document)(https://developers.cloudflare.com/pages/tutorials/build-an-api-with-workers/) for this exact purpose. I simply follow the first half of this document to build up my API. There are a few small issues you might run into if you try to follow the exact steps described in the document. Let me highlight them for you here.

Install wrangler
Setup datasource

