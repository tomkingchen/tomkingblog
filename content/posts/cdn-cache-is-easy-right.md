---
title: "CDN Cache is Easy, Right?"
date: 2026-09-06T16:47:00+10:00
draft: false
categories: "Field Notes"
description: "Why the same CDN hostname can hand back a different Cache-Control value for every single URL, and what that taught me about Akamai's Image and Video Manager."
---

TL;DR, CDN Cache is not easy.

Context: I have a S3 bucket for all my site's static contents. It's fronted by Akamai and served through `edge.tomking.xzy`.

It all started with a simple question: is `edge.tomoking.xyz` set to cache or not? But after one Akamai property lookup, the plot thickens: why does every single URL on this hostname have a different `Cache-Control` value?

Here's the windy road I took to find the answer in the end.

## The setup

First, I jump straight to Akamai console to check the "property" (Akamai's term for site configuration) cache configuration for `edge.tomoking.xyz`. It showed there's no single caching policy for this property. Instead, caching is decided per-rule based on file extension and hostname match, and different content types get routed through completely different logic.

## Finding #1: there is no one caching policy

The top-level `default` rule sets a baseline: `MAX_AGE`, TTL 90 days, `mustRevalidate: true`. But almost nothing on the hostname actually uses that baseline directly, it gets overridden by more specific child rules:

| Rule | Matches | Caching behavior |
|---|---|---|
| Static Content | css, js, mjs, html, pdf, txt, etc. | `CACHE_CONTROL_AND_EXPIRES` — **honors the origin's own headers** (private/no-store/no-cache/max-age), falls back to 90d only if origin sends nothing |
| JSON/XML Content | json, xml | Forced `0s` TTL — effectively never cached |
| Image and Video Manager | jpg, gif, jpeg, png | Forced `MAX_AGE 1d`, plus IVM image transformation (see below) |

So what have I learnt so far: asking "is this hostname cached?" is the wrong question. The right question is "which rule does *this specific file* match?" — and that depends on its extension, not the hostname.

## Finding #2: "honoring origin headers" means the CDN is only as consistent as the S3 bucket

For `.js`/`.css`/etc., Akamai isn't setting `Cache-Control` at all. It's relaying whatever the origin object has. I confirmed this by pulling two files straight from the S3 origin with `aws s3api head-object`:

```bash
aws s3api head-object --bucket tomo-static-assets-bucket --key tomotest/bad.png
{
    "ContentType": "image/png",
    "Metadata": {}
}
```

As you can see, no `CacheControl` field at all. Whatever value shows up at the edge for objects like this is either the 90-day fallback default, or for images something else entirely (see below)!

The one lesson that actually matters operationally: for static assets on this property, cache control is done at the origin side. The edge will faithfully reproduce whatever origin is set for every client.

## Finding #3: the PNG rabbit hole

OK, now we know how the Cache Control is configured. Next, I tested with two different PNG files hosted in the same S3 bucket. I just want to see how the Cache Control was set for those 2.

This is where it got interesting. Two PNGs, same hostname, wildly different behavior:

```bash
curl -i https://edge.tomoking.xyz/tomotest/bad.png
HTTP/2 200
server: Akamai Image Manager
content-type: image/jpeg
cache-control: private, no-transform, max-age=75828
```

```bash
curl -i https://edge.tomoking.xyz/landingpage/good.png
HTTP/2 200
server: AmazonS3
content-type: image/png
cache-control: max-age=83602
```

Both are `.png` files matching the exact same "Image and Video Manager" rule in the property config (`imageManager: applyBestFileType: true`). Yet one got reformatted to JPEG by Akamai and marked `private`; the other passed straight through from S3, untouched, as plain `image/png`.

After some digging, here's what I learnt: Akamai decided `jpeg` is a better format for my `bad.png` so it converted the png to jpeg on the fly, and this caused the cache-control to become `private`, since each client might receive its own version of the picture after the conversion.

If you would like to know more about this issue, here is the slop version from Claude:

---

**IVM (Image and Video Manager)** is Akamai's on-the-fly image optimization product. With `applyBestFileType` enabled, it analyzes each image's actual pixel content and decides, per-request, whether re-encoding to a more efficient format is worth it. This is completely independent of file extension or client `Accept` headers. `server: Akamai Image Manager` in the response tells you it actually ran the transform pipeline; `server: AmazonS3` tells you it passed the request straight through untouched.

My first guess for why one file got converted and the other didn't was "transparency" — `file` reported both as `8-bit/color RGBA`, so I assumed the one *without* real transparency got safely downgraded to JPEG (which has no alpha channel), while the one *with* transparency stayed PNG to avoid visibly breaking it.

That guess was half right, and worth correcting for anyone reading this later: **PNG's `RGBA` color type doesn't mean the image is actually using transparency.** Plenty of export tools always write an alpha channel whether anything is transparent or not. The real test is the alpha channel's *content*, not its presence:

```python
from PIL import Image
im = Image.open("bad.png").convert("RGBA")
im.getchannel("A").getextrema()
# → (255, 255)
```

Every pixel in `bad.png` has alpha = 255 — fully opaque, despite being RGBA-encoded. IVM inspects the actual pixel data (not just the color type header), determines there's nothing to lose, and safely re-encodes it as JPEG for the size win. The landing-page image, by contrast, most likely has real semi-transparent or transparent pixels somewhere, so IVM refuses to flatten it and serves the untouched original instead.

---

## Finding #4: what `Cache-Control: private` actually means (and doesn't)

The `bad.png` response above came back `private, no-transform, max-age=75828`, so what does the `private` actually mean.

`private` vs `public` is an instruction about **who is allowed to store and reuse the response downstream**:

- `private`: only the requesting client's own private cache (your browser) may store this. Any shared cache in the path, like a proxy, another CDN hop, or Akamai serving the exact same bytes to a different client, is told not to.
- `public`: any cache along the path, shared or private, can reuse it for anyone.

## Takeaways

So what have I learnt:

- Akamai Cache-Control is not as straightforward as it looked.
- Akamai CDN is not the only place to check for Cache-Control settings.
- File types/extensions are important on deciding the cache behaviour.
- Even for the same file type, the cache behaviour can be different.
