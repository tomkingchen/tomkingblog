---
title: 'Setup SSO Access to AWS Console with Azure AD'
date: 2018-06-21T23:05:00.002-07:00
draft: false
url: /2018/06/setup-sso-access-to-aws-console-with.html
---

  

As organization acquires more AWS accounts, it becomes quite a challenge for IT to manage the access to all those accounts. Instead of dealing with individual IAM accounts across multiple accounts. We need an identity solution to simplify the user access provision and removal process.

  

AWS itself offers a service called AWS SSO, which allows integrate AWS access with on premise AD through SAML. However, the service does incur charges and will require provision of an AD Connect appliance in AWS, if you don't already have ADFS in place(Yes, it has the same name as Azure AD Connect).

  

Instead of AWS SSO, in this article I will talk about how you can setup Azure AD as the sole IDP for all your AWS accounts through SAML. Azure AD as the core of Office 365 services is widely used across businesses these days. In a lot cases, organization has their on premise AD forests synced with Azure AD. This allows you to manage AWS access from on premise tools like ADUC. It also means you can take advantages like MFA from Office 365.

  

The solution requires NO additional infrastructure from either AWS or Azure, and comes with zero cost.

Microsoft has an [official guide](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-saas-amazon-web-service-tutorial) to show how to setup SSO between Azure AD and AWS. However, the configuration does require store the AWS root credential in Azure. The solution described below utilizes "App Roles" available in Azure App to define AWS access roles, therefore save the need of using the root account. 

  

To start, we setup Enterprise App in Azure AD.

*   Sign in to your Azure AD tenant as Global Administrator
    
*   Click on Azure AD → Enterprise applications → All applications → New Application → All
    
*   In the text box enter “AWS” and you should see 2 applications
    
*   Select the one with the black icon “Amazon Web Services (AWS) - Developer services”
    
*   Change the name as "AWS SSO", as it will display to end users, then click Add
    
*   Click on Single sign-on and select SAML-based Sign-on
    
*   Check the checkbox View and edit all other user attributes
    
*   Click on Add attribute and add the following
    

Name

Value

[https://aws.amazon.com/SAML/Attributes/RoleSessionName](https://aws.amazon.com/SAML/Attributes/RoleSessionName)

user.userprincipalname

[https://aws.amazon.com/SAML/Attributes/Role](https://aws.amazon.com/SAML/Attributes/Role)

user.assignedroles

*   Download the metadata XML file 
    
*   Click on Save
    

  

Next we need to setup Identity Provider in AWS.

*   Login to the AWS Console and click on IAM → Identity Providers → Create Providers
    
*   Select SAML as Provider Type
    
*   Enter "Contoso AzureAD" as Provider Name
    
*   Upload the Metadata XML file downloaded previously
    
*   Click on Next → Create
    
*   Copy the ARN of Identity Provider to Notepad for later use
    

  

Now we need to setup IAM Roles to define the access.

*   Click on Roles → Create new role → Select SAML 2.0 federation as the trusted entity type
    

           

[![](https://4.bp.blogspot.com/-Kgotfam37oE/WyyRT4hR_TI/AAAAAAAAKEw/0YEtTrb4WosbX_OblT3nAtGYzBYkMeEcQCLcBGAs/s320/saml.png)](https://4.bp.blogspot.com/-Kgotfam37oE/WyyRT4hR_TI/AAAAAAAAKEw/0YEtTrb4WosbX_OblT3nAtGYzBYkMeEcQCLcBGAs/s1600/saml.png)

*   Select "Contoso AzureAD" as SAML Provider
    
*   Click on Next Step
    
*   Select AdministratorAccess as policy
    
*   Enter "ContosoDev-Admins-SAML-AzureAD" as Role name. 
    
*   Copy the Role ARN to Notepad
    

  

Back in Azure AD, we now need to configure App Registrations.

*   Click on Azure AD → App Registrations
    
*   Select the "AWS SSO" application created. If you cannot find the app, make sure the search scope is set to "All Apps" instead of "My Apps".
    
*   Click the edit Manifest icon
    
*   Make sure you download a copy of the original Manifest file before carry out any changes
    
*   Edit the manifest by adding the following code to the appRoles array. The id has to be a unique number, which you can copy from an existing AppRole and modify it. Make sure "isEnabled" is "true", otherwise you won't be able to assign the role. For "value", simply copy the Role ARN and Identity Provider ARN from your Notepad.
    

            {"allowedMemberTypes": \[  
  
                "User"  
  
                \],  
  
            "displayName": "Contoso Dev AWS Admins",  
  
            "id": "7abcd756e-8c27-4472-b2b7-38c17ab51234",  
  
            "isEnabled": true,  
  
            "description": "ContosoDev-Admins-SAML-AzureAD",  
  
            "value": "arn:aws:iam::591818111111:role/ContosoDev-Admins-SAML-AzureAD,arn:aws:iam::591818111111:saml-provider/AzureAD"  
  
            },

*   Click Save to validate and save the file. If the validation failed, review your JSON format. You can use the downloaded copy as your recover point.
    

  

Now you are ready to assign AD groups to the AWS App.

*   Click on Azure AD → Enterprise applications → All applications
    
*   Select your application
    
*   Click on Users and Groups → Add user
    
*   Click on Users and groups
    
*   Select the AD group you created for AWS access, click Select
    
*   Click on Select Role
    
*   Select "Contoso Dev AWS Admin", click Select, click Assign
    
*   If you don't see the role here, go back to App Registration manifest file for AWS SSO and make sure "isEnabled" is set to true.
    

You can repeat above steps to setup SSO for additional roles and AWS accounts. 

  

Now if you go to [https://myapps.microsoft.com](https://myapps.microsoft.com/) and login as the group member of the AWS AD group, you should be able to see the AWS SSO app in the portal. Upon clicking the app, you will be redirected to the AWS Account if you only have one role and one account setup. Otherwise, you will see the options to choose between different roles under all your AWS accounts.

  

[![](https://3.bp.blogspot.com/-s-A0FKsnhhc/WyyRjPjZZ2I/AAAAAAAAKE0/kn2ygY1OKpwJptvqMCVMNxqnpny02qBCgCLcBGAs/s320/SSORoles.png)](https://3.bp.blogspot.com/-s-A0FKsnhhc/WyyRjPjZZ2I/AAAAAAAAKE0/kn2ygY1OKpwJptvqMCVMNxqnpny02qBCgCLcBGAs/s1600/SSORoles.png)

  

  

That's it! You have now setup AWS SSO across multiple accounts with your Azure AD identities.

  

But, hang on, there is one more thing... What about CLI and API access. With the default SSO setup, there is no option in AWS Console to generate Access Keys and Secrets. In the next blog, I will explain how we can get the necessary tokens through AWS SSO.

  

References:

[https://docs.microsoft.com/en-us/azure/active-directory/active-directory-saas-amazon-web-service-tutorial](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-saas-amazon-web-service-tutorial)

[http://blog.flux7.com/aws-best-practice-azure-ad-saml-authentication-configuration-for-aws-console](http://blog.flux7.com/aws-best-practice-azure-ad-saml-authentication-configuration-for-aws-console)