---
title: 'The Un-documented Way to Setup AWS SSO with Okta'
date: 2019-02-22T21:22:00.001-08:00
draft: false
url: /2019/02/the-un-documented-way-to-setup-aws-sso.html
---

In this article I would like to share an un-documented way of setting up AWS SSO by using Okta.In case you don’t know what Okta is. It is one of the popular identity management solutions out in the market. It provides Identity as a service through its Web portal and APIs.  
  
There is a detailed [document](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Amazon-Web-Service) provided by Okta walks through steps of how to setting up SAML SSO between your AWS accounts and Okta. So why don’t we follow that? Well, the solution suggested by the official document requires a privileged **USER Account** to be setup in your **Master AWS Account**. The account will then be configured to be able to assume roles in all your other accounts. In other words, if this account is compromised, your whole organization’s AWS Accounts are exposed to potential security breach.  
  
Ok, what’s the better way then? I actually talked about this in one of my old post [Setup SSO Access to AWS Console with Azure AD](https://www.tomking.xyz/2018/06/setup-sso-access-to-aws-console-with.html). Like the Azure AD solution, we will map user groups to different AWS Roles. With that in place, we can then assign different levels of AWS accesses to users by simply adding them into correspondent groups. This is actually a lot easier to setup in OKTA than Azure AD. Here’s how it can be done.  
  
**In OKTA**  
1\. Log into your OKTA tenant. If you don’t have one, you can sign up a free Developer account from [developer.okta.com](https://developer.okta.com/).  
2\. Click the Admin button on the up right corner  
  
3\. If you like me are using a Dev account, click Developer Console and switch to **Classic UI**.  
  
4\. Go to **Applications** and click **Add Application.**  
5\. Search for “Amazon Web Service”, NOT “AWS”, otherwise you will get a different app.  
  
6\. Add the application and leave everything as default and click Done. Don’t worry about the settings at this stage.  
  
7\. Now is the important step. After added your AWS application, raise a **Support Call** with OKTA! Yes, they provide support even for free dev accounts. And now ask for below specific attributes to be enabled for your AWS Application. They are hidden by default and can only be enabled by OKTA.  
  

*   APP FILTER
*   GROUP FILTER
*   ROLE VALUE PATTERN

  
The support call usually takes 48 hours to come around for a free dev account.   
  
8\. Go back to your AWS Application in OKTA, and click Edit under **Sign On** tab.  
9\. Change the **SIGN ON METHOD** to SAML 2.0.  
10\. Once the attributes are added, you should see something like below.  

[![](https://4.bp.blogspot.com/-JimaFQpljCM/XHDYRGyto1I/AAAAAAAAKd8/24qMY2ppvUE8Atuufi1oun-rGAChWMS3wCLcBGAs/s320/signonOptions.png)](https://4.bp.blogspot.com/-JimaFQpljCM/XHDYRGyto1I/AAAAAAAAKd8/24qMY2ppvUE8Atuufi1oun-rGAChWMS3wCLcBGAs/s1600/signonOptions.png)

  
The Group Filter applies to your group names in OKTA. The default Regex aws\_(?{{accountid}}\\d+)\_(?{{role}}\[a-zA-Z0-9+=,.@\\-\_\]+) means the group name needs to match something like “**aws\_12345678910\_adminrole**”. 12345..910 is the Account Number of the AWS Account.  
  
The Role Value Pattern applies to your role ARN in AWS. The Regex arn:aws:iam::${accountid}:saml-provider/OKTA,arn:aws:iam::${accountid}:role/${role} means the role ARN should be something like “**arn:aws:iam::12345678910:saml-provider/OKTA,arn:aws:iam::12345678910:role/adminrole**”. In this case, again 12345678910 refers your AWS Account number. OKTA is the name you gave to the Identity Provider in AWS.  
Both regex strings can be tweaked to fit your own needs.  
  
11\. Click the **Identity Provider metadata** link to download the metadata file.  

[![](https://3.bp.blogspot.com/-yZ7920U2TkY/XHDX2K0jgPI/AAAAAAAAKd0/P73raPcOep0JurTBMM8EKFi5eNKqIlNDACLcBGAs/s320/METAlink.png)](https://3.bp.blogspot.com/-yZ7920U2TkY/XHDX2K0jgPI/AAAAAAAAKd0/P73raPcOep0JurTBMM8EKFi5eNKqIlNDACLcBGAs/s1600/METAlink.png)

  
12\. Create OKTA groups to match the “Group Filter” regex string. Or, if your groups are synced from other identity source like AD, create the groups there. Assign AWS Application to the group(s).  
  
**In AWS**  
1\. Login to the AWS Console and click on IAM → Identity Providers → Create Providers  
2\. Select SAML as Provider Type  
3\. Enter "OKTA" as Provider Name  
4\. Upload the Metadata XML file downloaded previously  

> Click on Next → Create

> Copy the ARN of Identity Provider to Notepad for later use

Now we need to setup IAM Roles to define the access.  
5\. Click on Roles → Create new role → Select SAML 2.0 federation as the trusted entity type  
6\. Select "OKTA" as SAML Provider  
7\. Click on Next Step  
8\. Select AdministratorAccess as policy  
9\. Enter "awsadmin" as Role name.  
10\. Save all your changes.  
  
**Test**  
Back in OKTA, add a user to the “aws\_12345678910\_adminrole” group.  
  
Login as the user into OKTA. After user click the app, he/she should receive a login screen like below. In this case, the user has two roles associated with this particular account.  

[![](https://3.bp.blogspot.com/-lQNTtNIHytc/XHDXR_nrwqI/AAAAAAAAKds/GqFuk0CuASw-ukWL51bYQFoU7MPsrOZ5gCLcBGAs/s320/awsSSO.png)](https://3.bp.blogspot.com/-lQNTtNIHytc/XHDXR_nrwqI/AAAAAAAAKds/GqFuk0CuASw-ukWL51bYQFoU7MPsrOZ5gCLcBGAs/s1600/awsSSO.png)

  
  
The solution works for all scale of AWS environment. No matter how many AWS Accounts you have, all you need is user groups with matching name of its AWS Roles.  
  
The AWS Idp and role creation part can be further automated with CloudFormation or CLI scripts.  
In  compare to the Azure AD solution, there is no need to change any existing configuration (like the manifest JSON file in AAD) every time you add a role or whole new AWS account. Which is really convenient when you have like over 100 AWS accounts to manage!