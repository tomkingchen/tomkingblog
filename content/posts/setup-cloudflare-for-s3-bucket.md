---
title: 'Setup Cloudflare for S3 Bucket'
date: 2021-08-14T22:38:00.002-07:00
draft: false
url: /2021/08/setup-cloudflare-for-s3-bucket.html
---

One way to improve website performance is to use CDN to distribute the static contents of your site. S3 is a common place to host such type contents. In this post, I will show you how to publish a S3 bucket using Cloudflare. In fact, the screen shots used in this blogpost is published exactly through this manner.

Setup S3 Bucket Permissions
---------------------------

This is an optional step which adds a S3 bucket policy to your bucket. This restricts S3 access to  Cloudflare IPs only. It's still up to debate whether this is worth doing or not. But I believe it's always a best practice to reduce the public access surface whenever you can.

{ 

    "Version": "2012-10-17", 

    "Statement": \[ 

        { 

            "Sid": "PublicReadGetObject", 

            "Effect": "Allow", 

            "Principal": "\*", 

            "Action": "s3:GetObject", 

            "Resource": "arn:aws:s3:::YOUR\_BUCKET\_NAME/\*", 

            "Condition": { 

                "IpAddress": { 

                    "aws:SourceIp": \[ 

                        "2400:cb00::/32", 

                        "2606:4700::/32", 

                        "2803:f800::/32", 

                        "2405:b500::/32", 

                        "2405:8100::/32", 

                        "2a06:98c0::/29", 

                        "2c0f:f248::/32", 

                        "173.245.48.0/20", 

                        "103.21.244.0/22", 

                        "103.22.200.0/22", 

                        "103.31.4.0/22", 

                        "141.101.64.0/18", 

                        "108.162.192.0/18", 

                        "190.93.240.0/20", 

                        "188.114.96.0/20", 

                        "197.234.240.0/22", 

                        "198.41.128.0/17", 

                        "162.158.0.0/15", 

                        "172.64.0.0/13", 

                        "131.0.72.0/22", 

                        "104.16.0.0/13", 

                        "104.24.0.0/14" 

                    \] 

                } 

            } 

        } 

    \] 

}

  

To allow Internet visitors to access the contents, you do need to turn off Block all public access restriction. This of course is not recommended, if you host sensitive contents in the bucket other than just pictures and CSS. You should consider separate these contents into different buckets if this is the case.

[![](https://blogfiles.tomking.xyz/15-08-2021/s3publicblock.png)](https://blogfiles.tomking.xyz/15-08-2021/s3publicblock.png)

  

  

Change Request Header in Cloudflare
-----------------------------------

As indicated in this AWS document ([https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/VirtualHosting.html)), S3 only allows request headers with hostname set to its bucket name. In order to access the bucket using our own domain name (blogfiles.tomking.xyz), we have to either

*   Rename the bucket to blogfiles.tomking.xyz
*   Or, change the request header, so its hostname is changed to the S3 bucket name.

The drawback of option 1 is you will have to use HTTP as S3 static site does not support HTTPS. Plus you will have to rename URLs if the bucket name is changed. So to me, the preferred option is to change the header.

  

If you have a business account, this can be easily done by creating a **Page Rule**.

[![](https://blogfiles.tomking.xyz/15-08-2021/pagerule.png)](https://blogfiles.tomking.xyz/15-08-2021/pagerule.png)

  

  

But if you are poor like me using a free account, then we will have to create a Worker to do this. Below is the code to achieve the same as the Page Rule.

async function handleRequest(request) {

  // Override host header with S3 URL 

  let newurl = new URL(request.url);

  newurl.hostname = "YOUR\_BUCKET\_NAME.s3.ap-southeast-1.amazonaws.com";

  const newRequest = new Request(

    newurl,

    request,

   )

   try {

     return await fetch(newRequest)

   } catch (e) {

     return new Response(JSON.stringify({ error: e.message }), { status: 500 })

   }

 }

 addEventListener("fetch", event => {

  event.respondWith(handleRequest(event.request))

})

  

Create a CNAME Record for the Bucket
------------------------------------

Depends on where you host the DNS record for the bucket, these are the steps to cutover the traffic to Cloudflare.

### DNS hosted in Cloudflare

1.  Create a CNAME record as below, which points the subdomain to the S3 bucket.
2.    
    
    [![](https://blogfiles.tomking.xyz/15-08-2021/cname.png)](https://blogfiles.tomking.xyz/15-08-2021/cname.png)
    
      
    

### DNS not hosted in Cloudflare

1.  Create a CNAME record in **Cloudflare** as shown above.
2.  Create a CNAME record in **your own DNS provider** as below, which points the subdomain to Cloudflare DNS server.

blogfiles 300 IN CNAME blogfiles.tomking.xyz.cdn.cloudflare.net.

  

That's it, now you should be able to browse the S3 contents using the custom domain name, which is going through Cloudflare CDN.

A useful test is using cURL command below. As you can see it got a 200 response back and confirmed it's a MISS, and server is cloudflare .

➜ ~ curl https://blogfiles.tomking.xyz/29-07-2021/cloudflare.png -v

\* Trying 104.21.12.143...

\* TCP\_NODELAY set

\* Connected to blogfiles.tomking.xyz port 443 (#0)

\* ALPN, offering h2

\* ALPN, offering http/1.1

\* successfully set certificate verify locations:

...

\* Using Stream ID: 1 (easy handle 0x55cb8d261580)

\* TLSv1.3 (OUT), TLS Unknown, Unknown (23):

\> GET /29-07-2021/cloudflare.png HTTP/2

\> Host: blogfiles.tomking.xyz

\> User-Agent: curl/7.58.0

\> Accept: \*/\*

\>

...

< HTTP/2 200

< date: Sun, 15 Aug 2021 05:27:06 GMT

...

< cf-cache-status: MISS

...

< server: cloudflare

...

\* Connection #0 to host blogfiles.tomking.xyz left intact