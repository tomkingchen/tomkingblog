---
title: 'On Premise Mailbox user missing in Exchange Online GAL'
date: 2015-11-01T20:12:00.002-08:00
draft: false
url: /2015/11/on-premise-mailbox-user-does-not-show.html
tags: 
- GAL
- Exchange Hybrid
- Office 365
- Cloud
---

Ran into an interesting issue with one of our Exchange Online customer. Thought it probably worth sharing with the solution I found.  
  
The customer has an Exchange Hybrid setup. Recently some of  Office 365 Exchange Online users complain they cannot email to a particular on premise mailbox: [Paul.Smith@contoso.com](mailto:Paul.Smith@contoso.com). The user bascialy does not show up in Exchange Online GAL. The on premise mailbox is working fine and other on premise staff can send emails to it without issue. When I checked Exchange Online, I cannot find the contact object for this account.  

  

Initially I thought it could be for some reason the user's on premise AD object has not been synced properly to Office 365. So I tried with recreating the object in Office 365 by moving the on premise AD account to a non-synced OU. This delete the user's Office 365 account. Then I move the on premise account back into its original OU. After re-sync the account back into Office 365, it still refuse to show up in Exchange Online contact list.

  

As always, I contacted Microsoft support to try get an answer. But that didn't prove to be too useful either. The suggestion MS provided is to "Change our Email domain **contoso.com** from Authority to Internal Relay". This issue only affects a single user out of 500 and MS is suggesting to change a Global setting. I give them up quickly and focus on research the issue myself.

  

I then took a close look at the user's Office 365 account, while compare it with other accounts I notice a difference in the field of **CloudExchangeRecipientDisplayType**. 

Get-MSOLUser –UserPrincipalName _paul.smith@contoso.com_ | fl > c:\\paul.txt

Get-MSOLUser –UserPrincipalName _tom.leigh@contoso.com_ | fl > c:\\tom.txt

  

**Paul's user details**

![](file:///C:/Users/chent/AppData/Local/Temp/enhtmlclip/Image(21).png)

[![](http://3.bp.blogspot.com/-N2nK4e3yCJY/VjLAxmgWt7I/AAAAAAAAJKg/8HdHpZ62UUo/s320/pauldetail.png)](http://3.bp.blogspot.com/-N2nK4e3yCJY/VjLAxmgWt7I/AAAAAAAAJKg/8HdHpZ62UUo/s1600/pauldetail.png)

  

  

  

  

**Another user details**

[![](http://1.bp.blogspot.com/-YeM7H3u7D3I/VjLBDL5g_HI/AAAAAAAAJKo/V_sT-LIQQW0/s320/userdetail.png)](http://1.bp.blogspot.com/-YeM7H3u7D3I/VjLBDL5g_HI/AAAAAAAAJKo/V_sT-LIQQW0/s1600/userdetail.png)

  

![](file:///C:/Users/chent/AppData/Local/Temp/enhtmlclip/Image(22).png)

  

  

  

According to this [article](http://blogs.technet.com/b/johnbai/archive/2013/09/11/o365-msexchangerecipienttypedetails.aspx), -2147483642 means "SynedMailboxUser". This is the recipient type of Paul's Office 365 account. So now we know what's wrong with Paul's account.

  

To fix the issue, we just need to set **CloudExchangeRecipientDisplayType** with the correct value. To do so we need to make change to Paul's on premise AD account.

  

In on premise ADUC, enable **Advanced Features** in view. Open the user properties and select **Attribute Editor** tab.

  

Change following fields

**msExchRecipietnDisplayType -> 6**

**msExchRecipientTypeDetails -> 128**

**targetAddress -> paul.smith@contoso.com**

  

Make sure you take a screenshot of the existing value

![](file:///C:/Users/chent/AppData/Local/Temp/enhtmlclip/Image(23).png)

![](file:///C:/Users/chent/AppData/Local/Temp/enhtmlclip/Image(24).png)

  

After change the value, perform a AD SYNC from DirSync or run **Start-OnlineCoexistenceSync**. 

  

In the SYNC log, confirm the user object has been updated. If you go into the Exchange Online contact list, Paul's email address should be available now.

  

Once confirmed user email address available in Office 365 again, we need to reverse the above change to the on premise user account. Otherwise, Paul will not be able to access his on premise mailbox.

  

To do so, simply apply the following changes

**msExchRecipietnDisplayType -> 1073741824**

**msExchRecipientTypeDetails -> 1**

  

**targetAddress -> <not set>**