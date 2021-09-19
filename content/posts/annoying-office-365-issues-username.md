---
title: 'Annoying Office 365 issues: Username won''t sync through DirSync'
date: 2014-06-18T18:03:00.001-07:00
draft: false
url: /2014/06/annoying-office-365-issues-username.html
---

It happens to me quite a few times. After synced On-Premise AD user objects into Office365 through DirSync, users' specified UPN will not be synced correctly into Office 365.  
  
Tried all the suggested tricks from Office 365 forum. None of them worked. I even tried to re-configure our DirSync settings. But the username in O365 just refuse to change. After banging my head on this issue for a few days. Eventually I figured it out, the solution is simply to revoke any O365 licenses assigned to the user. Then conduct a full sync with "Start-OnlineCoexistenceSync". With licenses allocated to the user, Office 365 cannot change the account's UPN.  
  

[![](http://3.bp.blogspot.com/-69Bjg7NwmVs/U6I1MnKGLMI/AAAAAAAAJF0/QZSAeomoFME/s1600/nolicenses.png)](http://3.bp.blogspot.com/-69Bjg7NwmVs/U6I1MnKGLMI/AAAAAAAAJF0/QZSAeomoFME/s1600/nolicenses.png)

  
  
  
  
  
  
  
  
  
  
  
  
It may be necessary to change the username AFTER revoke all the O365 licenses and resync. DirSync doesn't seem to pickup the difference between the On-Premise AD object and the O365 Azure AD object. It only upload whatever attribute is changed.