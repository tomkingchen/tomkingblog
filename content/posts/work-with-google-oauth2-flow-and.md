---
title: 'Use PowerShell to delete SPAM Blogger comments'
date: 2021-02-27T21:29:00.004-08:00
draft: false
url: /2021/02/work-with-google-oauth2-flow-and.html
---

I haven't been very diligent on maintaining this blog. There has been quite a few SPAM comments accumulated on my posts. I am going to turn on moderation to block those. But I need a way to clean all those existing SPAM comments. So over the weekend, I wrote this PowerShell script to do just that. 

In the end, it will probably take less time if I just manually all the cleanup manually. But I believe this is a good opportunity to learn about Google API and OAuth2 flow. Plus, now I have something to write about :)

Google API can be accessed with both API key and OAuth2 token. However, the API key is only allowed for public accessible resources and actions like read a post or comment. Actions like deleting a post or comment require OAuth2 to be setup. Now let's look at how this is achieved using PowerShell.

### **Setup the app in Google Developer Console**

Go to https://console.developers.google.com/ and setup a new project.

Under **Library**, find **Blogger v3 API** and enable it.

Under **OAuth consent screen**, create new consent screen for your project. 

Fill in only the compulsary options, leave the rest and continue.

For scopes, choose the Blogger API v3 blogger scope.

[![](https://1.bp.blogspot.com/-yJf9SHvViB4/YDseOaH0PTI/AAAAAAAAKzA/_bz5Zs1MCk8uxWQLKkS-UqZ13A8M92csgCLcBGAsYHQ/s320/image.png)](https://1.bp.blogspot.com/-yJf9SHvViB4/YDseOaH0PTI/AAAAAAAAKzA/_bz5Zs1MCk8uxWQLKkS-UqZ13A8M92csgCLcBGAsYHQ/s480/image.png)

  

  

  

  

  

In next screen, don't add any test users and just click **Save and Continue**.

Review the summary and save.

Once the consent screen is created, we can now create OAuth2 Client ID.

For Client ID, all you need to know is that for PowerShell scripts, we will use **Desktop app** as the app type.

Save the Client ID and Client Secret in a secure vault like LastPass or BitWarden.

### Get OAuth2 Token using PowerShell

I will skip explaining about OAuth2, as it's quite a large topic and I have already done so in my previous post [here](https://www.tomking.xyz/2018/11/understand-oauth-and-open-id-connect.html). Instead, I will walk you through the PowerShell code in details.

To obtain the access token, we need following parameters.

*   Client ID and Client Secret of your app/script, which we created in the console.
*   Scope of your app (https://www.googleapis.com/auth/blogger)
*   Google Authorization Endpoint (https://accounts.google.com/o/oauth2/v2/auth)
*   Google Token Endpoint (https://www.googleapis.com/oauth2/v4/token)

With these information, we can then form a proper request URI as below:

Start-Process "https://accounts.google.com/o/oauth2/v2/auth?client\_id=$client\_id&scope=$scopes&access\_type=offline&response\_type=code&redirect\_uri=urn:ietf:wg:oauth:2.0:oob"

This will initialize an interactive browser session using the supplied URI. Authorize the request with your Google account and you will get the **Authorization Code** in the end.

The code can then be stored as a variable using **Read-Host**.

We then use **Invoke-WebRequest** to send a POST request to the URL below. 

https://www.googleapis.com/oauth2/v4/token?client\_id=$client\_id&client\_secret=$client\_secret&redirect\_uri=urn:ietf:wg:oauth:2.0:oob&code=$code&grant\_type=authorization\_code

This request will return both the **Access Token** and **Refresh Token**. Normally the access token is only valid for an hour. For our script, this is way more than enough. We will not worry abou the refresh token.

### Access Blogger API v3 using OAuth2 Token

The access token should be used as a parameter in the API request in the format below.

access\_token=your\_access\_token

In addition to this, we will need following information.

Blog ID, this can be easily identified via your Blogger URL.

Blogger API URI, which is https://blogger.googleapis.com/v3/blogs.

A typical request URI to Blogger API looks like this:

https://blogger.googleapis.com/v3/blogs/$blogid/posts/$postid/comments/$commentid?access\_token=your\_access\_token

### Script Logic

With the access to API sorted, the rest are pretty straight forward. I simply check posts with reply field set to more than 0. Then list all the comments of those posts. If the comment contains hyper-link syntax "href=", it will be categorized as a SPAM. The script will show all these type of comments and prompt me to confirm whether they should be deleted or not. Upon receiving confirmation, the script then sends a DELETE request to each comments API URI to remove them.

You can find more details about how to form the query from my code:

[https://github.com/tomkingchen/blogger-cleaner](https://github.com/tomkingchen/blogger-cleaner)