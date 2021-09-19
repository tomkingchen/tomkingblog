---
title: 'OWA and ECP failure after Install Exchange 2016 CU17 '
date: 2020-11-02T21:07:00.005-08:00
draft: false
url: /2020/11/owa-and-ecp-failure-after-install.html
---

I recently ran into an issue after update Exchange 2016 from CU15 to CU17. The upgrade installation took around an hour, but was eventually completed successfully according to the Installation Wizard at least. When I tried to access ECP, I got the error below even before the login page shows up. At the meantime, Exchange Management Shell is inaccessible due to the error.

[![](https://lh3.googleusercontent.com/-gd0qSlqsU_8/X6DdzzhtAhI/AAAAAAAAKto/4N8H88608cAzJ56dDwd0oNtALMzAi6xtQCLcBGAsYHQ/image.png)](https://lh3.googleusercontent.com/-gd0qSlqsU_8/X6DdzzhtAhI/AAAAAAAAKto/4N8H88608cAzJ56dDwd0oNtALMzAi6xtQCLcBGAsYHQ/image.png)

  
In the eventlog, there are lots of 1003 errors relate to MSExchange Front End HTTP Proxy. 

[![](https://lh3.googleusercontent.com/-9LuMl4xt0u8/X6DeCvAr6_I/AAAAAAAAKts/p-KDCvE_sKgrkvqF1aVI2EIaVYYmingvACLcBGAsYHQ/image.png)](https://lh3.googleusercontent.com/-9LuMl4xt0u8/X6DeCvAr6_I/AAAAAAAAKts/p-KDCvE_sKgrkvqF1aVI2EIaVYYmingvACLcBGAsYHQ/image.png)

After some research, it appears the issue is caused by corrupted SharedWebConfig.config files in C:\\Program Files\\Microsoft\\Exchange Server\\V15\\FrontEnd\\HttpProxy and C:\\Program Files\\Microsoft\\Exchange Server\\V15\\ClientAccess. But regenerating the files base on [this MS document](https://docs.microsoft.com/en-us/exchange/troubleshoot/client-connectivity/event-1309-code-3005-cannot-access-owa-ecp) didn't fix the issue.

I end up had to setup a test Exchange 2016 CU17 in my own lab environment. Once the new Exchange is up, I copied those 2 SharedWebConfig.config files to the production Exchange server and then did a IISRESET. To my dismay, ECP this time came up with the login page, but after authentication it returned back with a blank page.

Upon checking eventlog, I notice there are bunch of HttpEvent errors. These errors seem to indicate issue is to do with the certificate used for ECP and OWA.

[![](https://lh3.googleusercontent.com/-M3V3TkH3wf8/X6DjTAPxgqI/AAAAAAAAKuI/RBT2fskckWoQJytJdXv4C1E4EAvGUG6eACLcBGAsYHQ/image.png)](https://lh3.googleusercontent.com/-M3V3TkH3wf8/X6DjTAPxgqI/AAAAAAAAKuI/RBT2fskckWoQJytJdXv4C1E4EAvGUG6eACLcBGAsYHQ/image.png)

It turns out the ECP backend was somehow binded to a deleted certificate. So the fix is just to set the backend binding to a valid SSL certificate and restart IIS!

I am still not sure how those SharedWebConfig files are corrupted, as the CU installation process completed successfully. But if you need, here is [the link](https://github.com/tomkingchen/exchange2016cu17sharedwebconfig) for the good CU17 config files. Save you the hassle to build another Exchange server (It took me hours).