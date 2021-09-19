---
title: 'Package and deploy a PowerShell Lambda function with custom modules'
date: 2020-06-06T05:20:00.003-07:00
draft: false
url: /2020/06/package-and-deploy-powershell-lambda.html
tags: 
- PowerShell
- AWS
---

#### Very helpful guide! I'll note that a little mo...
[cole patrol](https://www.blogger.com/profile/05931720336991755692 "noreply@blogger.com") - <time datetime="2021-05-22T13:41:33.683-07:00">May 0, 2021</time>

Very helpful guide! I'll note that a little more background on the creation of the module itself would be handy. What I've found after following this guide:  
\- AWS requires you to create a .psd1 in the root directory of your module. Make sure the RootModule parameter in this .psd1 points to your module .psm1 file, like so: "RootModule = 'mymodule.psm1'"  
\- If you have multiple functions in this module, make sure you are either listing them as an array or using a wildcard to export them all (though wildcards are discouraged for performance reasons)  
\- FunctionsToExport = '\*'  
Same goes with any variables you might want to export from your modules script:  
\- VariablesToExport = '\*'  
  
Good luck! Setting this up is admittedly muchh more difficult than python/nodejs environments in lambda
<hr />
