---
title: 'Migrate Azure AD Connect Between AD Forests'
date: 2018-05-10T04:22:00.000-07:00
draft: false
url: /2018/05/migrate-azure-ad-connect-between-ad.html
---

#### I have done multi-forest several times, but recent...
[Dustin](https://www.blogger.com/profile/09539881205956772897 "noreply@blogger.com") - <time datetime="2020-08-01T13:32:35.147-07:00">Aug 0, 2020</time>

I have done multi-forest several times, but recently starter multi-forest as part of a forest migration (other times had multi-forest as an end-state). The old server, I didn't intend on establishing multi-forest with that server. However, We setup a new AAD Connect server connected to the new forest, and then added the old forest. Really weird, but the connector for the new forest kept trying to authenticated with the service account it created for the old forest even though the connector had the proper account in the MIISClient and when reviewing the configuration with the ADSync PowerShell module. I think we will uninstall it and add the new forest to the existing server and do the full forest migration, then step back through adding the new server.  
  
Just to illustrate it a bit more, the service account that Azure AD Connect makes is MSOL\_. The account is the same name as it is derived from a hash of the server name (is my understanding). But, it is a separate account in each forest and it is listed as having the forest/domain name associated with their respective forests (e.g. OLD.FOREST\\MSOL\_ and NEW.FOREST\\MSOL\_).
<hr />
#### Well... that didn't work as expected. I have ...
[Dustin](https://www.blogger.com/profile/09539881205956772897 "noreply@blogger.com") - <time datetime="2020-08-01T13:35:28.575-07:00">Aug 0, 2020</time>

Well... that didn't work as expected. I have tags in there that were labeled Hash... anytime you see MSOL\_, think MSOL\_\[Hash\].  
  
Also, the error was that the service account for OLD.FOREST\\MSOL\_\[Hash\] does not have permissions to the domains in NEW.FOREST. We could potentially get around it all by granting those permissions, but that definitely isn't what we want to do as OLD.FOREST is going away.
<hr />
#### I have the same scenario where its a need to move ...
[Surinderpal Singh](https://www.blogger.com/profile/17756010976509417302 "noreply@blogger.com") - <time datetime="2021-01-26T12:22:04.490-08:00">Jan 3, 2021</time>

I have the same scenario where its a need to move Azure AD connect from one forest to another, the challenge that I see is the AD domain is same as public domain on new forest which I believe making it a bit complex to go for as this AD domain of new forest is authorized domain on Exchange server in old forest.  
  
xyz.com (new forest) <=> abc.com (old forest) - having Exchange on old forest with xyz.com as authoritative domain.  
  
Not sure how to handle it with above process except the one to break AD connect on old forest and then setup on new forest and do hard match with accounts.  
  
Please advise.
<hr />
#### Cant you setup a temp alternative UPN suffix on th...
[Tom](https://www.blogger.com/profile/14494757221262153880 "noreply@blogger.com") - <time datetime="2021-01-28T15:17:06.110-08:00">Jan 5, 2021</time>

Cant you setup a temp alternative UPN suffix on the new forest?
<hr />
#### Hello Tom, Thanks for your response. I thought ab...
[Surinderpal Singh](https://www.blogger.com/profile/17756010976509417302 "noreply@blogger.com") - <time datetime="2021-02-01T11:32:28.756-08:00">Feb 2, 2021</time>

Hello Tom,  
Thanks for your response.  
  
I thought about it, username to login will change with this step and later on when getting rid of it, user login will be changed back on 3565 to use xyz.com.  
  
I was thinking here to setup Azure AD sync and leave the email addresses on xyz.com and have each forest domain as authorized on O365, both abc.com and xyz.com. Since trust is present, it should also work. Your thoughts.
<hr />
