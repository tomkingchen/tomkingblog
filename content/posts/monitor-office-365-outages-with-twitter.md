---
title: 'Monitor Office 365 Outages with Twitter'
date: 2018-07-28T05:29:00.000-07:00
draft: false
url: /2018/07/monitor-office-365-outages-with-twitter.html
tags: 
- Office 365
---

Office 365 has high SLAs ([the latest English version](http://www.microsoftvolumelicensing.com/Downloader.aspx?DocumentId=14033)) backed by Microsoft’s excellent Azure Cloud. However, like every other cloud services, there is always the chance for something unexpected to happen.

This year April Office 365 had a major hiccup to its service. Its Asia Pacific backend Azure AD authentication went haywire. As a result, users lost access all O365 services. To make it worse, the usual Office 365 monitoring channel: Office 365 dashboard was not accessible due to this fault.

To track the issue at the time, I ended up relying on the official MS Twitter account @Office365Status for updates. The account contains all Office 365 service outage notifications. This made it become the only source for people to track the issue at that time.

After the incident, it become obvious that the Twitter account is a pretty reliable source for monitoring Office 365 service status. Based on this, I developed a solution, it uses PowerShell script to check for new tweets from @Office365Status Twitter account. The scheduled script is executed from a AWS EC2 instance, this completely eliminates the dependence from Microsoft services.

Ok, let’s go through the script in details.

In order to allow our script to read tweets, we first need to create an app in [https://apps.twitter.com](https://apps.twitter.com). Please be aware the site will be consolidated into [https://developer.twitter.com](https://developer.twitter.com) soon. And as of July 2018, you will need to apply for a Twitter developer account in order to create a new app.

[![image](https://lh3.googleusercontent.com/-uoIQj0dqUBQ/W1xhnQuSt9I/AAAAAAAAKIs/GdlEnFR79P4jYnGYlmyZ_mFeWNFSwsPAwCHMYCw/image_thumb%255B1%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-HMX0MrR4vyM/W1xhl1-KbaI/AAAAAAAAKIo/eYtJlzwhwRMcu855dDSodhxWdzq-4ytegCHMYCw/s1600-h/image%255B3%255D)

After created the app, under Permissions, set the permission as Read only, AS the monitoring script will have no need to write.

[![image](https://lh3.googleusercontent.com/-D-DA5sebsmg/W1xlqxc3-7I/AAAAAAAAKJM/TQYiMVT2NvsXHY22zGPMNA2Bbs1H_qyywCHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-7qbo8_iMsy0/W1xlp8_roII/AAAAAAAAKJI/0yxQF2JvjvYvF8z8yrzi_l3eu6LZK26_wCHMYCw/s1600-h/image%255B6%255D)

Under Keys and Access Tokens, copy following key values to notepad. These values will be needed in the script to configure the access to Twitter.

*   Consumer Key (API Key)
*   Consumer Secret (API Secret)
*   Access Token
*   Access Token Secret

Now we just need a PowerShell module that uses Twitter API to pull out tweets. Among all the PowerShell modules I tested, MyTwitter, written by Adam Bertram is the only one works for me. You can find the module from [here](https://gallery.technet.microsoft.com/scriptcenter/Tweet-and-send-Twitter-DMs-8c2d6f0a).

First, we create a authenticated session with the generated app tokens.

```
New-MyTwitterConfiguration \-APIKey YourTwitterAppAPIKey \`  
\-APISecret YourTwitterAppAPISecret \`  
\-AccessToken YourTwitterAppAccessToken \`  
\-AccessTokenSecret YourTwitterAppAccessTokenSecret  

```

Next, we use Get-TweetTimeline to get recent tweets from @Office365Status account. Then we grab the first item from the array, which is the latest tweet from the account.

```
$TimeLine \= Get-TweetTimeline \-UserName "office365status" \-MaximumTweets 100   
$tweet \= $TimeLine\[0\]   

```

The script will then get the creation time of the latest tweet. Twitter uses a non standard time format. So first, we will need to convert the time to a conventional format. This will allow us to compare the creation time with the current time on the server.

Twitter original creation time format:

 [![image](https://lh3.googleusercontent.com/-ZDQBtBveKF0/W12X5ehaIuI/AAAAAAAAKKQ/OXF6CGtomH8e4gl3gVo4juRrbQ1TG9d2gCHMYCw/image_thumb%255B5%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-d5C9qmNRzeY/W12X39ttQtI/AAAAAAAAKKM/66PB9LDr3A0ed0taIexstfyI0yewAhsjgCHMYCw/s1600-h/image%255B11%255D)

To convert the time we use a Convert-DateString function.

```
$createTime \= Convert-DateString \-Date $tweet.created\_at \`
``````
\-Format 'ddd MMM dd HH:mm:ss zzzz yyyy'  

```

ddd = First 3 letter of weekday.

MMM = First 3 letter of month.

dd = day of the month

HH:mm:ss = hours:minutes:seconds

zzzz = time zone hours

yyyy = year

The function uses .Net method TryParseExact to convert the time format to something like below.

[![image](https://lh3.googleusercontent.com/-ijXZuR9Re_c/W12X7tgzLiI/AAAAAAAAKKY/EdOmprvEeQYekzQu7MI9aZTwT4pB5HhDACHMYCw/image_thumb%255B6%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-12vIAsjFFQU/W12X6kad5eI/AAAAAAAAKKU/-GNbRmhKcTwrBma5Q4hpDLd3jBJpTfNkACHMYCw/s1600-h/image%255B14%255D)

```
function Convert-DateString (\[String\]$Date, \[String\[\]\]$Format)  
{  
   $result \= New-Object DateTime  
   
   $convertible \= \[DateTime\]::TryParseExact(  
      $Date,  
      $Format,  
      \[System.Globalization.CultureInfo\]::InvariantCulture,  
      \[System.Globalization.DateTimeStyles\]::None,  
      \[ref\]$result)  
   
   if ($convertible) { $result }  
}  

```

Once we have the correct creation time, my script will check if the tweet is a reply. This is to filter out those reply tweets. As we only care about the incident updates.

The script will then check if the tweet is created within last hour. Depends on the interval you schedule the script. You may set this to the frequent you desire. In my case, I schedule the script to check every hour, hence the 1 hour interval.

Upon confirm the tweet is posted within last hour and is not a reply tweet, the script will then send out an email to a monitoring mailbox (Not your Office 365 mailbox of course!).

```
if ($tweet.in\_reply\_to\_user\_id\_str \-like ""){  
    \# If the tweet is created within last hour, send out alert  
    if ($createTime \-gt (get-date).Addhours(-1)){  
          
        $body \= $tweet.text  
        $Emailsubject \= "Microsoft Office365Status Twitter Account just post a new update"  
        Send-MailMessage \-from "tom.chen@contoso.com" \-To "it@contoso.com" \-SmtpServer mailserver.contoso.com \-Subject $Emailsubject \-Body $body  
    }else{  
        return  
    }  
}  

```

The PowerShell monitoring script can be downloaded from my GitHub repo [here](https://github.com/tomkingchen/PowershellScripts/blob/master/MonitorO365Status_pub.ps1).

The last part of the solution is to upload the script to a server (EC2 in AWS in my case) and schedule it to run every hour!

Below is an Email I received for a recent incident. Hope you find this useful!

[![image](https://lh3.googleusercontent.com/-Qa6eKiG_uMw/W12X-Og0L2I/AAAAAAAAKKg/fGzwcS_QbEo4zhtzcKhKZteFUv3U-OYRgCHMYCw/image_thumb%255B7%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-11H9StkvGaA/W12X84cg3ZI/AAAAAAAAKKc/GGfUZ6VM63kOJXtMy5jtMQO16Nzl4IIuwCHMYCw/s1600-h/image%255B17%255D)