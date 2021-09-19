---
title: 'Setup AWS SSO CLI & API Access'
date: 2018-06-25T17:24:00.001-07:00
draft: false
url: /2018/06/setup-aws-sso-cli-api-access.html
---

  

In my last article, I discussed the steps to setup AWS SSO  through Azure AD. By using Azure AD app roles, we are able to use our Azure AD accounts to access AWS Console. But with this measure, you will find there is no option in AWS IAM to generate Access Key and Secrete for CLI and API access.

  

Fortunately, we are not the only ones out there have this problem. David T Johnson faced the same issue and he is kind and smart enough (unlike me) to create a tool to address this issue. The tool source code can be found on Github [https://github.com/dtjohnson/aws-azure-login](https://github.com/dtjohnson/aws-azure-login)

  

The tool is written in Node.Js. So if you don't have Node, the first thing will be to install Node from [https://nodejs.org/en/](https://nodejs.org/en/). After that simply follow the installation instructions to get the tool going. 

  

The example show here is tested from Windows 10. 

  

Before start, log into Azure portal to get Azure Tenant ID and the AWS SSO App ID.

The tenant ID can be found in Properties section of Azure AD. The Directory ID is basically your Tenant ID.  

[![](https://2.bp.blogspot.com/-W_iJiXwELG0/WzGFgUnr_RI/AAAAAAAAKFU/0fpRAs8TCY4BaN19Fr7jnOzASs1HAfNDwCLcBGAs/s400/directoryid.png)](https://2.bp.blogspot.com/-W_iJiXwELG0/WzGFgUnr_RI/AAAAAAAAKFU/0fpRAs8TCY4BaN19Fr7jnOzASs1HAfNDwCLcBGAs/s1600/directoryid.png)

  

In Azure AD, go to App Registrations and select the AWS SSO app, if you don't see the AWS SSO app, change the scope to "All Apps".  

[![](https://3.bp.blogspot.com/-JbWxtW2J7PE/WzGFyk0jauI/AAAAAAAAKFc/JqLnsXqi308eDmJpyEfVSqBlTRk-bXEUACLcBGAs/s400/awsapp.png)](https://3.bp.blogspot.com/-JbWxtW2J7PE/WzGFyk0jauI/AAAAAAAAKFc/JqLnsXqi308eDmJpyEfVSqBlTRk-bXEUACLcBGAs/s1600/awsapp.png)

  

Click Settings -> Properties, copy the App ID URI.

[![](https://2.bp.blogspot.com/-dVFKPkQ9OpU/WzGGAnH35qI/AAAAAAAAKFg/bDp4PazR-W8Bk0EUQ9V8JLe3iIC8NgFowCLcBGAs/s320/appid.png)](https://2.bp.blogspot.com/-dVFKPkQ9OpU/WzGGAnH35qI/AAAAAAAAKFg/bDp4PazR-W8Bk0EUQ9V8JLe3iIC8NgFowCLcBGAs/s1600/appid.png)

  

In the Properties section, copy the App ID URI.

  

Open Node.js command prompt

[![](https://4.bp.blogspot.com/--44256V9T_s/WzGGQdv9UdI/AAAAAAAAKFo/zWtlSf-eDbwt1oKZSB7gZ1TeCqfoCTGIACLcBGAs/s320/nodecmd.png)](https://4.bp.blogspot.com/--44256V9T_s/WzGGQdv9UdI/AAAAAAAAKFo/zWtlSf-eDbwt1oKZSB7gZ1TeCqfoCTGIACLcBGAs/s1600/nodecmd.png)

  

Setup the authentication profile

   

[![](https://3.bp.blogspot.com/-9YQ4rtDXorE/WzGGeD8cBmI/AAAAAAAAKFw/eHsrFV7g6Rc-7PyDb2Ga8VjJi_S2xhg-wCLcBGAs/s400/authprofile.png)](https://3.bp.blogspot.com/-9YQ4rtDXorE/WzGGeD8cBmI/AAAAAAAAKFw/eHsrFV7g6Rc-7PyDb2Ga8VjJi_S2xhg-wCLcBGAs/s1600/authprofile.png)

  

 Sign in with the profile, select the role in each accounts. It will be a lot easier to tell which account the roles belong to, if you named your roles with the AWS Account name. E.g. ContosoDev-Admins-SAML -AzureAD.

[![](https://2.bp.blogspot.com/-eldJHICpoWw/WzGGpj3l5PI/AAAAAAAAKF4/tpDVEWvo9VMeNpqT6_bIcOgik50wOd0RwCLcBGAs/s400/samlsignin.png)](https://2.bp.blogspot.com/-eldJHICpoWw/WzGGpj3l5PI/AAAAAAAAKF4/tpDVEWvo9VMeNpqT6_bIcOgik50wOd0RwCLcBGAs/s1600/samlsignin.png)

  

The session duration needs to be changed in your AWS role definition first. By default it is set to 1 hour only. So if you keep getting errors when setting duration longer than 1 hour, please follow this article to change the API duration. [https://aws.amazon.com/blogs/security/enable-federated-api-access-to-your-aws-resources-for-up-to-12-hours-using-iam-roles/](https://aws.amazon.com/blogs/security/enable-federated-api-access-to-your-aws-resources-for-up-to-12-hours-using-iam-roles/)

[![](https://4.bp.blogspot.com/--V5L0ImASlM/WzGG22yIPRI/AAAAAAAAKGA/eo_XHYW-MF03vetk5F9lqjMnSY8Dnbr5gCLcBGAs/s400/iamroles.png)](https://4.bp.blogspot.com/--V5L0ImASlM/WzGG22yIPRI/AAAAAAAAKGA/eo_XHYW-MF03vetk5F9lqjMnSY8Dnbr5gCLcBGAs/s1600/iamroles.png)

  

After finish the sign-in process, go to .aws subfolder and you should see a credentials file there. The file contains all the existing profiles and the STS keys and tokens. 

[![](https://3.bp.blogspot.com/-YeOtTolJGgU/WzGHCU4oXvI/AAAAAAAAKGI/w9-6GcJGVn4UUtc-MyRPxx3g7QxaRqP6ACLcBGAs/s400/keys.png)](https://3.bp.blogspot.com/-YeOtTolJGgU/WzGHCU4oXvI/AAAAAAAAKGI/w9-6GcJGVn4UUtc-MyRPxx3g7QxaRqP6ACLcBGAs/s1600/keys.png)

  

You can then use the keys to access AWS resources via CLI and API calls. Below is an example from AWS Toolkit for Visual Studio.

  

AWS Toolkit for Visual Studio can be downloaded from [https://aws.amazon.com/visualstudio/](https://aws.amazon.com/visualstudio/)

  

Create a new AWS template project in Visual Studio, and open AWS Explorer. Choose "contosoaws" as the profile.

[![](https://3.bp.blogspot.com/-3owxaDalaOs/WzGHWDCZLdI/AAAAAAAAKGQ/ALZ3aK49s1YELN5s4gTO6yrl9yrA4-w-wCLcBGAs/s400/awsexpl.png)](https://3.bp.blogspot.com/-3owxaDalaOs/WzGHWDCZLdI/AAAAAAAAKGQ/ALZ3aK49s1YELN5s4gTO6yrl9yrA4-w-wCLcBGAs/s1600/awsexpl.png)

  

With the STS credential, you should see all the instances in the region.

[![](https://1.bp.blogspot.com/-qVmikO_JliY/WzGHtC2p6qI/AAAAAAAAKGc/PXbLtOMZnvUFhpAjkNqy5O-X_fgzB8jxgCLcBGAs/s400/instances.png)](https://1.bp.blogspot.com/-qVmikO_JliY/WzGHtC2p6qI/AAAAAAAAKGc/PXbLtOMZnvUFhpAjkNqy5O-X_fgzB8jxgCLcBGAs/s1600/instances.png)