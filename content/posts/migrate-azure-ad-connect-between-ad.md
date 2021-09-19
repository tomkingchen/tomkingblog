---
title: 'Migrate Azure AD Connect Between AD Forests'
date: 2018-05-10T04:22:00.000-07:00
draft: false
url: /2018/05/migrate-azure-ad-connect-between-ad.html
---

  
Migrate Azure AD Connect Between AD Forests  

  

I was recently involved in an AD forest migration project for one of our customers. As part of the requirements, we need to move the existing AD Connect server to a newly created AD forest. While the process itself is pretty straight forward, I do notice there aren't many online resource out there detail the whole process. So to make things even easier for the folks out there, I will share the steps I took to complete this AD Connect migration.

  

The details shown here are from my sandpit environment. 

The existing/source AD forest: [old.contoso.com](http://old.contoso.com/)

The existing AD DC/AD Connect server: old-dc1.old.contoso.com

The new/target AD forest: [new.contoso.com](http://new.contoso.com/)

The new AD DC/AD Connect server: [new-dc1.new.contoso.com](http://new-dc1.new.contoso.com/)

UPN Suffix: [briwave.com](http://briwave.com/)

  

Here's a simple diagram to help you visualize the whole setup.

[![](https://2.bp.blogspot.com/-MBe5BCiGRP4/WvQqOqpv_LI/AAAAAAAAKB0/i2IyGMQf6N8wEN7USlo3Vz4OAM4MEuE3QCLcBGAs/s400/adconnectmigdiagrm.png)](https://2.bp.blogspot.com/-MBe5BCiGRP4/WvQqOqpv_LI/AAAAAAAAKB0/i2IyGMQf6N8wEN7USlo3Vz4OAM4MEuE3QCLcBGAs/s1600/adconnectmigdiagrm.png)

  

**Add New AD Forest to Existing AD Connect**

  

First, we need to add the new forest [new.contoso.com](http://new.contoso.com/)into the existing AD Connect sync scope.

Below are the steps to do so:  
  
1\. On old-dc1.old.contoso.com, open AD Connect. Under the Task, choose Customize synchronization options.

2\. Enter the Azure AD Global Admin credential to connect.

3\. Add the new AD forests into AD Connect.

[![](https://2.bp.blogspot.com/-9Ovfmq16xeo/WvQq0jrRY3I/AAAAAAAAKB8/MOtA2jXY0bkB59KNZtEZUw7_qUrYBnqwgCLcBGAs/s400/confirmnewad.png)](https://2.bp.blogspot.com/-9Ovfmq16xeo/WvQq0jrRY3I/AAAAAAAAKB8/MOtA2jXY0bkB59KNZtEZUw7_qUrYBnqwgCLcBGAs/s1600/confirmnewad.png)

4\. Confirm the UPN Suffix is verified.

5\. Record the OU Filtering settings for the forest. These settings will need to be applied to the new AD Connect later.  

  

[![](https://4.bp.blogspot.com/-et7Ze8mlOcA/WvQrI_fj-lI/AAAAAAAAKCE/MwzjfYOf17gDDvyiVOhi8MAjjTfiQsu0ACLcBGAs/s400/OUfiltering_old.png)](https://4.bp.blogspot.com/-et7Ze8mlOcA/WvQrI_fj-lI/AAAAAAAAKCE/MwzjfYOf17gDDvyiVOhi8MAjjTfiQsu0ACLcBGAs/s1600/OUfiltering_old.png)

  

6\. Record the settings here as well for later configuration on new-dc1.new.contoso.com.  

[![](https://2.bp.blogspot.com/-IGhcRbpOvzs/WvQrJAl32bI/AAAAAAAAKCI/SlBdFUVaFKYPTrrT0ATQtY9CeshJ4PwfQCEwYBhgL/s400/option_old.png)](https://2.bp.blogspot.com/-IGhcRbpOvzs/WvQrJAl32bI/AAAAAAAAKCI/SlBdFUVaFKYPTrrT0ATQtY9CeshJ4PwfQCEwYBhgL/s1600/option_old.png)

7\. Click Configure to apply the changes.  
8\. Click Exist to close.  
  

To verify the configuration, open Synchronization Service on OLD-DC1 after run start-ADSyncSyncCycle and you should be able to see bunch of actions for [new.contoso.com](http://new.contoso.com/)forest. If you check Azure Portal, you should also be able to see the newly provisioned users and groups from [new.contoso.com](http://new.contoso.com/)AD.

[![](https://4.bp.blogspot.com/-MAtvQdL0nq4/WvQrg_kAtrI/AAAAAAAAKCU/K4LWNXgtJYwMIZibaIaQBoXjZSUnhwRqgCLcBGAs/s400/syncresult_old.png)](https://4.bp.blogspot.com/-MAtvQdL0nq4/WvQrg_kAtrI/AAAAAAAAKCU/K4LWNXgtJYwMIZibaIaQBoXjZSUnhwRqgCLcBGAs/s1600/syncresult_old.png)

  

* * *

**Install AD Connect in Target AD Forest**

The next step is to install and configure AD Connect in the new forest.

1\. After download the AD Connect onto new-dc1.new.contoso.com, choose Custom install. 

2\. Click **Install** to proceed.

3\. Depend on the sign on method your organization use, you will need to choose the one accordingly. For my lab, I simply use the default password hash sync.

4\. Put in Azure AD Global admin account to connect.

5\. Add both forests to the scope.  

[![](https://2.bp.blogspot.com/-L76X_H6cKnY/Wvq_hiXc7OI/AAAAAAAAKCw/p2uTlDEXIX4KzEt9eN_OmGZAHmGdZpsCQCLcBGAs/s400/syncbothad.png)](https://2.bp.blogspot.com/-L76X_H6cKnY/Wvq_hiXc7OI/AAAAAAAAKCw/p2uTlDEXIX4KzEt9eN_OmGZAHmGdZpsCQCLcBGAs/s1600/syncbothad.png)

  

6\. Confirm UPN suffix and username option.  

[![](https://2.bp.blogspot.com/-E_6MR7i2s8g/Wvq_tnsjujI/AAAAAAAAKC0/0_mNpsCs3PYlB6OFQuQxTLvLDxGJOoT7gCLcBGAs/s400/confirmupn.png)](https://2.bp.blogspot.com/-E_6MR7i2s8g/Wvq_tnsjujI/AAAAAAAAKC0/0_mNpsCs3PYlB6OFQuQxTLvLDxGJOoT7gCLcBGAs/s1600/confirmupn.png)

  

7\. Select OU Filter for both forests, make sure it mirrors the current settings on Old-dc1

8\. Choose Users are represented only once across all directories, as we will migrate the users across to the new forest later. With Source anchor, unless you use non default AD attribute for GUID, leave it unchanged.  

[![](https://3.bp.blogspot.com/-L-QJlB44SME/Wvq_-cgeEOI/AAAAAAAAKDA/3vm3Fynoqkg9dSj7xLLvx1MnrEIYUPoxgCLcBGAs/s400/uniqueiduser.png)](https://3.bp.blogspot.com/-L-QJlB44SME/Wvq_-cgeEOI/AAAAAAAAKDA/3vm3Fynoqkg9dSj7xLLvx1MnrEIYUPoxgCLcBGAs/s1600/uniqueiduser.png)

  
9\. Leave this unchanged, unless you use group based filtering.

10\. With the optional features, you will want to keep the same settings as old-dc1 to avoid any unexpected issues.

11\. Choose Enable staging mode... this will ensure that the AD Connect will not sync after complete the setup.  

[![](https://3.bp.blogspot.com/-Et1c-QXfgrw/WvrAM0PFtSI/AAAAAAAAKDE/6D6jxjHQTLUdu2nTTg2_eWilF816k2FnwCLcBGAs/s400/enablestaging.png)](https://3.bp.blogspot.com/-Et1c-QXfgrw/WvrAM0PFtSI/AAAAAAAAKDE/6D6jxjHQTLUdu2nTTg2_eWilF816k2FnwCLcBGAs/s1600/enablestaging.png)

  

That's it. Click **Install** to finish the setup.

To confirm the staging mode setup, you can run start-ADSyncSyncCycle and check the log in Synchronization Service. As you can see below, there is no Export action to Azure.  

[![](https://2.bp.blogspot.com/-qEidzOAp2cI/WvrAfqPW1FI/AAAAAAAAKDQ/be-0b0t0FDErRb88BMg-Kya51er9dd13wCLcBGAs/s400/noexport.png)](https://2.bp.blogspot.com/-qEidzOAp2cI/WvrAfqPW1FI/AAAAAAAAKDQ/be-0b0t0FDErRb88BMg-Kya51er9dd13wCLcBGAs/s1600/noexport.png)

  

* * *

**Switch Sync Source From Old to New**

The next step is to switch the synchronization from the old AD Connect to the new one.

To do this, we first turn AD Connect on OLD-DC1 to Staging mode.

Open AD Connect on OLD-DC1 and choose **Configure staging mode (current state: disabled)**

[![](https://1.bp.blogspot.com/-AJXc1ExQ3qw/WvrAz358GZI/AAAAAAAAKDY/QNv15k2dgD0tSQWjjkb5ThsH41M7eS91wCEwYBhgL/s400/enablestagingonold.png)](https://1.bp.blogspot.com/-AJXc1ExQ3qw/WvrAz358GZI/AAAAAAAAKDY/QNv15k2dgD0tSQWjjkb5ThsH41M7eS91wCEwYBhgL/s1600/enablestagingonold.png)

  

Tick the checkbox to enable **Staging** mode.

Uncheck **Start the synchronization process when configuration completes**. If you leave this checked, AD objects from OLD forest will still be synced to AD Connect metaverse(database), but will not be exported to Azure AD.

[![](https://3.bp.blogspot.com/-5YEPYd9peUI/WvrBP_HuZRI/AAAAAAAAKDg/hbBA4314cjwd4JFQJ64S-WLxgZShr_RfACLcBGAs/s400/unchecksync.png)](https://3.bp.blogspot.com/-5YEPYd9peUI/WvrBP_HuZRI/AAAAAAAAKDg/hbBA4314cjwd4JFQJ64S-WLxgZShr_RfACLcBGAs/s1600/unchecksync.png)

  

Once click **Configure**, the AD Connect will proceed to enable staging mode and stop sync to Azure AD.

Next, we disable staging mode on NEW-DC1. 

On NEW-DC1, launch AD Connect and select **Configure staging mode (current state: enabled)**

[![](https://4.bp.blogspot.com/-vZK2Akb0T7o/WvrBc1OIOvI/AAAAAAAAKDk/bLFeqYR5VbQU1yy_iBVBwr7NkIzICq6XACLcBGAs/s400/disablestaging.png)](https://4.bp.blogspot.com/-vZK2Akb0T7o/WvrBc1OIOvI/AAAAAAAAKDk/bLFeqYR5VbQU1yy_iBVBwr7NkIzICq6XACLcBGAs/s1600/disablestaging.png)

  

Put in Azure AD credential and uncheck **Enable staging mode.**  
  

[![](https://4.bp.blogspot.com/-BPdmo4bmZFQ/WvrBxNTJBmI/AAAAAAAAKDw/yOHF5vsQUc8Vfm5FFExB6eThH2X8kIz5gCLcBGAs/s400/uncheckstaging.png)](https://4.bp.blogspot.com/-BPdmo4bmZFQ/WvrBxNTJBmI/AAAAAAAAKDw/yOHF5vsQUc8Vfm5FFExB6eThH2X8kIz5gCLcBGAs/s1600/uncheckstaging.png)

Check **Start the synchronization process when configuration completes.**

Click **Configure** to apply the changes.  
  

  

**Test AD Sync from NEW-DC1**

  

Run **Start-ADSyncSyncCyle Initial** to kick off a AD Sync from NEW-DC1. From the result below you can see it picks up an update.

[![](https://2.bp.blogspot.com/-c5EQoA1eCa4/WvrB7BQFrWI/AAAAAAAAKD0/ZoF4rUz-GgA5FqzLrEED9a81OaTyAV48gCLcBGAs/s400/synctestresult.png)](https://2.bp.blogspot.com/-c5EQoA1eCa4/WvrB7BQFrWI/AAAAAAAAKD0/ZoF4rUz-GgA5FqzLrEED9a81OaTyAV48gCLcBGAs/s1600/synctestresult.png)

  

In the properties view of the update item, you can see **ms-DS-ConsistencyGUID** of **Tony Zhang** in **NEW.contoso.com** has now been added to Azure AD**.** It wasn't before because the user was just created when AD Connect in **OLD.contoso.com** did the sync.

[![](https://2.bp.blogspot.com/-XK4fEcPEUGo/WvrCEzN_83I/AAAAAAAAKD8/f-k8ZjfyoekrFr04_J9Da5xl9xoQLVrhQCLcBGAs/s400/syncoutput.png)](https://2.bp.blogspot.com/-XK4fEcPEUGo/WvrCEzN_83I/AAAAAAAAKD8/f-k8ZjfyoekrFr04_J9Da5xl9xoQLVrhQCLcBGAs/s1600/syncoutput.png)

  

So that's it. As simple as that, we have completed the migration of AD Connect from one AD forest to another!