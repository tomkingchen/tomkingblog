---
title: 'Setup Cloudflare for AWS API Gateway'
date: 2021-06-27T00:25:00.003-07:00
draft: false
url: /2021/06/setup-cloudflare-for-aws-api-gateway.html
---

 In this post I will show how to setup Cloudflare for a Serverless app built with AWS API Gateway and Lambda. For demonstration, I use a simple web app I built (rona.tomking.xyz). The app is hosted in  AWS Sydney region. It displays daily Victoria COVID case and that's it.

  

To use Cloudflare, I have signed up a free Cloudflare account. The **first site** can be added for free with following features.

*   DNS hosting
*   Free SSL certificates
*   CDN
*   DDoS attacks mitigation up to 67 Tbps capacity
*   Up to 100k workers requests and 30 scripts
*   3 Page rules

  

To allow Cloudflare become the first ingress point for our web app, the first step is to change our DNS Nameserver to Cloudflare's DNS NS records. In Cloudflare portal, click \+ Add site

, and type the site TLD, in our case it's tomking.xyz. Cloudflare will display the NS records set for the site. As I got my tomking.xyz from GoDaddy, I then need to log into GoDaddy DNS Zone management portal and change the nameservers from GoDaddy's NS records to Cloudflare's NS. By default Cloudflare will pickup all existing records, so you shouldn't need to recreate the records already there.

[![](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/nameservers.png)](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/nameservers.png)

  

  

Once confirm the NS change worked successfully, the next step is to create a certificate using AWS ACM. The certificate will be used by API Gateway Custom Domain to verify the redirection.

*   In ACM, click **Request a certificate** and choose to **Request a public certificate**.
*   Add your site's domain name to the certificate. In my case it is rona.tomking.xyz.
*   Choose **DNS validation** and add a Name tag to the certificate.
*   Submit the certificate request and you should be given a TXT record to be added to your DNS zone.
*   Add the TXT record in Cloudflare and wait for ACM to validate the record. This should take no longer than an hour.

After the certificate status changed to Issued you can now setup **Custom domain name** for the API Gateway.

*   In API Gateway, under **Custom domain name** click **Create** to create a new custom domain.
*   Type in the domain name, which is rona.tomking.xyz for the demo.
*   Use **TLC 1.2** as recommended TLS version.
*   Choose the ACM certificate we have just created and click **Create domain name**.
*   Under API mapping select the API you want to associate with the custom domain name.
*   Record the API Gateway domain name for the mapped custom domain.

[![](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/endpointconfig.png)](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/endpointconfig.png)

Come back to Cloudflare DNS portal, add a CNAME record for the mapped API gateway.

[![](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/cname.png)](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/cname.png)

  

  

That's it! Try hit the URL a few times and you should see traffic coming through Cloudflare and show up under **Analytics** tab.

[![](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/analytic.png)](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/analytic.png)

  

  

Look at the certificate of rona.tomking.xzy, it's using Cloudflare SNI certificate.

[![](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/snicert.png)](https://tomking-blog-files.s3.ap-southeast-2.amazonaws.com/27-06-2021/snicert.png)