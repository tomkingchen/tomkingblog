---
title: 'Receive error: Target mailbox doesn''t have an SMTP proxy matching ''.mail.onmicrosoft.com'' when move mailbox to Office 365'
date: 2014-04-06T23:51:00.005-07:00
draft: false
url: /2014/04/office-365-exchange-hybrid-deployment.html
tags: 
- Exchange Hybrid
- mail.onmicrosoft.com
- Mailbox Migration
- Office 365
---

Recently I was working on Exchange Hybrid Deployment for one of our customer. The Hybrid Configuration process itself went smoothly. No errors for HCW.   
  
The problems came when I tried to move mailbox to Exchange Online. The migration fails with error:  
**Target mailbox doesn't have an SMTP proxy matching '<domain>.mail.onmicrosoft.com'"**  
  
To check that I ran the following command against the On-premise Exchange.  
  
Get-Mailbox "O365 Test5" | fl and check the EmailAddress filed  

[![](http://3.bp.blogspot.com/-xfnTdyJyULk/U0JKuLIyF0I/AAAAAAAAJCI/N_bl6fxO_0o/s1600/getmailbox.png)](http://3.bp.blogspot.com/-xfnTdyJyULk/U0JKuLIyF0I/AAAAAAAAJCI/N_bl6fxO_0o/s1600/getmailbox.png)

  
  

  
Then I ran get-mailuser in Exchange Online. Because the mailbox hasn't been migrated, the user still show up as Mailuser(Contact) in Office 365.  
  
Get-MailUser "o365 test6" |fl  
  
  

[![](http://4.bp.blogspot.com/-uIrsas0ytfw/U0JINlFHZDI/AAAAAAAAJB0/mxOOdYZj5S4/s1600/getmailuser.png)](http://4.bp.blogspot.com/-uIrsas0ytfw/U0JINlFHZDI/AAAAAAAAJB0/mxOOdYZj5S4/s1600/getmailuser.png)

  
  
Noticed the proxy address "contoso.mail.onmicrosoft.com" does not show up into the Azure AD object. The user object in Office 365  only has our primary SMTP address "username@contoso.com" and "username@contoso.onmicrosoft.com" stamped. "CONTOSO.MAIL.onmicrosoft.com" is missing from the address list.  
  
After review the DirSync rules, I notice for user object, the proxyAddress attribute does not have an "Import" rule associated with it. As a test I manually added a Import rule for "proxyAddresses".  
The rule details are:  

cd.all:proxyAddresses->mv.all:proxyAddresses

  

[![](http://1.bp.blogspot.com/-sV_2gnEWJ54/UzpSrP0C_LI/AAAAAAAAJBU/JshPC8DHPjU/s1600/dirsync+rule.png)](http://1.bp.blogspot.com/-sV_2gnEWJ54/UzpSrP0C_LI/AAAAAAAAJBU/JshPC8DHPjU/s1600/dirsync+rule.png)

  

After kicked off a full sync by run "Start-OnlineCoexistenceSync".   
  
"get-mailuser" was able to return with the SMTP addresses "contoso.mail.onmicrosoft.com". Mailbox migration to Office 365 went successfully after that.