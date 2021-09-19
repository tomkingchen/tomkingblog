---
title: 'Use Microsoft Graph API to extract Excel file contents'
date: 2018-12-09T02:40:00.000-08:00
draft: false
url: /2018/12/use-microsoft-graph-api-to-extract.html
---

Recently I was working on automating a process that extracts contents from a Excel spreadsheet stored in Office 365 SharePoint Online. It took me quite sometime to figure out how to get this done. I thought there must be people out there looking for the same thing. Hence here is the post.

Based on the requirements, the Excel file needs to be shared among few specified staff. They should be able to modify the spreadsheet with their Office 365 accounts. Then a automated process will export the contents of the sheet to an external destination.

The first half of the request can be easily achieved with Office 365 Group. While creating Office 365, it generates a SharePoint site for the team (Group Drive), which allows members of the team to share documents within the group. For the second half, we will be using MS Graph along with PowerShell. Microsoft Graph API provides the ability to read and modify Excel workbooks stored in OneDrive for Business, SharePoint site or Group drive, which is exactly the place we will store the Excel file.

The first thing we need to do is to create a Office 365 Group. In this example I created one named as "ShareTest". This will in turn creates a SharePoint site dedicated for the group members to share files. The site URL is https://contoso.sharepoint.com/sites/sharetest.

[![Image(9)](https://lh3.googleusercontent.com/-6pUnRvAp8RI/XAzwzwpbq1I/AAAAAAAAKac/qc3_pExUGEAlgQlLWCDp1xc5BexfjynFwCHMYCw/Image%25289%2529_thumb?imgmax=800 "Image(9)")](https://lh3.googleusercontent.com/-rfZ8HSBAVKw/XAzwy2D-QHI/AAAAAAAAKaY/biuMf-DdnHc7KZI9LK9o-JN_xhfTuYT7gCHMYCw/s1600-h/Image%25289%2529%255B3%255D)

As you can see an Excel file is uploaded to the site. Any members of the "ShareTest" group can now read this document by default. To allow the members to edit the file, click the permission drop down buttion and select Edit. This will give all group members right to edit any files upload to the site.

[![Image(10)](https://lh3.googleusercontent.com/-7kpAF_GKs9A/XAzw2GTQjjI/AAAAAAAAKak/RoVaRiyAuswp_lTPEHNDZno-y0HF5KsrQCHMYCw/Image%252810%2529_thumb?imgmax=800 "Image(10)")](https://lh3.googleusercontent.com/-MkDoadrj8uM/XAzw1c-pWcI/AAAAAAAAKag/U3u9vv0qknUT2KsY6iXPaH2O9z9CaoAFACHMYCw/s1600-h/Image%252810%2529%255B3%255D)

Now we have solved our first requirement, next we need to identify the MS Graph URI to retrieve the Excel contents. There is a [Microsoft doucment](https://docs.microsoft.com/en-us/graph/api/resources/excel?view=graph-rest-beta) explains how to use MS Graph to work with Excel spreadsheets. But there is no mentioning about how to get the contents if the Excel file is stored in SharePoint. After some researching, I finally find the way to identify URI. Here are the steps I took.

1.  Log into MS Graph Explorer with a ShareTest member account.
    
2.  We created the site with 365 group. So the URI we use is for **groups** instead of **sites**. The URI uses query parameters to search for the group started with "sharetest". It will return the 365 Group details, which contains the group id of the 365 group.
    

**Note:** More about query parameters can be found [here](https://docs.microsoft.com/en-us/graph/query-parameters).

https://graph.microsoft.com/v1.0/groups/?$filter=startswith(displayname,'sharetest')

[![Image(11)](https://lh3.googleusercontent.com/-zaxuRRGLUNc/XAzw4c0yjyI/AAAAAAAAKas/gdRUPfr1hw4Xhc3s0Y6wou4WW1fG26MLQCHMYCw/Image%252811%2529_thumb?imgmax=800 "Image(11)")](https://lh3.googleusercontent.com/-YASqQgxkBOY/XAzw3OXkJUI/AAAAAAAAKao/xOEmRb_LgPsa3QQXDEDvtFPeV0GM77bhgCHMYCw/s1600-h/Image%252811%2529%255B3%255D)

3.  With the group id, we can now form the URI to query the SharePoint site. The URI will list all files in the library.
    

https://graph.microsoft.com/v1.0/groups/{group-id}/drive/root/children

[![Image(12)](https://lh3.googleusercontent.com/-qz1LhhNpBSA/XAzw58yr1PI/AAAAAAAAKa0/H8j2Xz5OeRQ2bcwA-8-FbBF31V7KT8wkgCHMYCw/Image%252812%2529_thumb?imgmax=800 "Image(12)")](https://lh3.googleusercontent.com/-dNbYBxno1Fs/XAzw5Caj1hI/AAAAAAAAKaw/v-pcUD83EcciYd2JrosbpYvkMRLO1bUJwCHMYCw/s1600-h/Image%252812%2529%255B3%255D)

4.  Next, we need to identify the worksheet ID in order to export its contents.
    

https://graph.microsoft.com/v1.0/groups/{group-id}/drive/**root:/example.xlsx:**/workbook/worksheets

[![Image(13)](https://lh3.googleusercontent.com/-YqOBueF4qzk/XAzw7qEUFNI/AAAAAAAAKa8/nuB27it4AdI794vVHy0-S3M1A7IPkzQ8wCHMYCw/Image%252813%2529_thumb?imgmax=800 "Image(13)")](https://lh3.googleusercontent.com/-19NuimhfSB8/XAzw6qU6cHI/AAAAAAAAKa4/tvWjcOgRFlUCh5DOaJR8dHYT_MLEgLOWwCHMYCw/s1600-h/Image%252813%2529%255B3%255D)

5.  This is the final URI we need to get the contents out. We use the range function to include all the data in the sheet.
    

https://graph.microsoft.com/v1.0/groups/{group-id}/drive/root:/example.xlsx:/workbook/worksheets('{worksheet-id}')/Range(address='Sheet1!A1:G10')

[![Image(14)](https://lh3.googleusercontent.com/-aPySzgTX_0g/XAzw9g_x2tI/AAAAAAAAKbE/IbUs9Gf9IVE2ta8SBGhiCvO2u3TlBWuOQCHMYCw/Image%252814%2529_thumb?imgmax=800 "Image(14)")](https://lh3.googleusercontent.com/-3uY6393a1_4/XAzw8ux-5xI/AAAAAAAAKbA/BO7_m4Uxv9I6BTQd5gGvfrw9c8XJu-FzwCHMYCw/s1600-h/Image%252814%2529%255B3%255D)

6.  The output is a JSON output of the worksheet contents. It still needs a lot work to strip away those unnecessary bits. But if you copy paste the whole contents into http://json2table.com, you will get something like below.
    

[![Image(15)](https://lh3.googleusercontent.com/-Al7hYp3VYnU/XAzxABHecoI/AAAAAAAAKbM/gLTuU9u71O44CcE-8jpPFaLxdXpUyDD-ACHMYCw/Image%252815%2529_thumb?imgmax=800 "Image(15)")](https://lh3.googleusercontent.com/-J_ItylqYH1s/XAzw-ij23DI/AAAAAAAAKbI/H1tMiGdObQAagKa10syD_L2O2qpceSbmwCHMYCw/s1600-h/Image%252815%2529%255B3%255D)

The above process can be automated with a PowerShell script. But to do that, first we need to register an Azure AD App for the script.

Log into your Azure portal, to create the Azure AD App, you do not need a subscription. So just go to **Azure Active Directory** and under **App registration**, click **\+ New application registration**.

The process to create the app is pretty straight forward. Give the app a proper name, like "PowerShell App". And make sure **Application Type** is set to: **Native**. **Redirect URI** does not need to be a real URL. So just leave it as https://redirectURI.com.

[![Image(16)](https://lh3.googleusercontent.com/-ZdtwICoAjec/XAzxCnncRhI/AAAAAAAAKbU/rJUHB2kmZNcbFLkA0ODkuaPxo_lVdA4egCHMYCw/Image%252816%2529_thumb?imgmax=800 "Image(16)")](https://lh3.googleusercontent.com/-mkWTNbDYsmA/XAzxBthRjSI/AAAAAAAAKbQ/yg7ZODi6ffsJXeYJTslVIMoLcnKo77gHACHMYCw/s1600-h/Image%252816%2529%255B3%255D)

Once the app is created, go to **Settings** and click **Required permissions**.

[![Image(17)](https://lh3.googleusercontent.com/-wi4Rg2snhF4/XAzxEpzFr4I/AAAAAAAAKbc/Erd2zbrH0PcHyk-nlDmBFBwKu1boI05rQCHMYCw/Image%252817%2529_thumb?imgmax=800 "Image(17)")](https://lh3.googleusercontent.com/-CHKAZEuK39c/XAzxDnklfVI/AAAAAAAAKbY/BYe9VYkJUBsuiM6sOxdjJ-dYRViDScq_QCHMYCw/s1600-h/Image%252817%2529%255B3%255D)

Add following Microsoft Graph Delegated Permissions to the app. This will allow the PowerShell script to query the API with proper delegated rights.

*   Read files that the user selects (preview)
    
*   Read user files
    
*   Read all files that user can access
    

[![Image(18)](https://lh3.googleusercontent.com/-jw7mkT_Uojs/XAzxGf3l6fI/AAAAAAAAKbk/OfAc_BoehbwvVZRvRE-_Sg9VKPwPouXmACHMYCw/Image%252818%2529_thumb?imgmax=800 "Image(18)")](https://lh3.googleusercontent.com/-7IxeNwfoVgQ/XAzxFnzBi_I/AAAAAAAAKbg/AXo-QIWcTXYUUm9r-4ZnFRglkbGJyeTagCHMYCw/s1600-h/Image%252818%2529%255B3%255D)

Once you save the changes, write down the **Application ID.**

Here is the script to extract contents from the Excel spreadsheet.

```
Function Get-AccessToken ($TenantName, $ClientID, $redirectUri,  $resourceAppIdURI, $CredPrompt){  
  
    Write-Host "Checking for AzureAD module..."  
  
    if (!$CredPrompt){$CredPrompt \= 'Auto'}  
  
    $AadModule \= Get-Module \-Name "AzureAD" \-ListAvailable  
  
    if ($AadModule \-eq $null) {$AadModule \= Get-Module \-Name "AzureADPreview"  \-ListAvailable}  
  
    if ($AadModule \-eq $null) {write-host "AzureAD Powershell module is not  installed. The module can be installed by running 'Install-Module AzureAD' or  'Install-Module AzureADPreview' from an elevated PowerShell prompt. Stopping." \-f  Yellow;exit}  
  
    if ($AadModule.count \-gt 1) {  
  
        $Latest\_Version \= ($AadModule | select version | Sort-Object)\[-1\]  
  
        $aadModule      \= $AadModule | ? { $\_.version \-eq $Latest\_Version.version  }  
  
        $adal           \= Join-Path $AadModule.ModuleBase  "Microsoft.IdentityModel.Clients.ActiveDirectory.dll"  
  
        $adalforms      \= Join-Path $AadModule.ModuleBase  "Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"  
  
        }  
  
    else {  
  
        $adal           \= Join-Path $AadModule.ModuleBase  "Microsoft.IdentityModel.Clients.ActiveDirectory.dll"  
  
        $adalforms      \= Join-Path $AadModule.ModuleBase  "Microsoft.IdentityModel.Clients.ActiveDirectory.Platform.dll"  
  
        }  
  
    \[System.Reflection.Assembly\]::LoadFrom($adal) | Out-Null  
  
    \[System.Reflection.Assembly\]::LoadFrom($adalforms) | Out-Null  
  
    $authority          \= "https://login.microsoftonline.com/$TenantName"  
  
    $authContext        \= New-Object  "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext"  \-ArgumentList $authority  
  
    $platformParameters \= New-Object  "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters"     \-ArgumentList $CredPrompt  
  
    $authResult         \= $authContext.AcquireTokenAsync($resourceAppIdURI,  $clientId, $redirectUri, $platformParameters).Result  
  
    return $authResult  
  
    }  
  
Function Invoke-MSGraphQuery($AccessToken, $Uri, $Method, $Body){  
  
    Write-Progress \-Id 1 \-Activity "Executing query: $Uri" \-CurrentOperation  "Invoking MS Graph API"  
  
    $Header \= @{  
  
        'Content-Type'  \= 'application\\json'  
  
        'Authorization' \= $AccessToken.CreateAuthorizationHeader()  
  
        }  
  
    $QueryResults \= @()  
  
    if($Method \-eq "Get"){  
  
        do{  
  
            \# $Results =  Invoke-RestMethod -Headers $Header -Uri $Uri  -UseBasicParsing -Method $Method -ContentType "application/json"  
  
            $Results \=  Invoke-RestMethod \-Headers $Header \-Uri $Uri \-Method  $Method  
  
            if ($Results.value \-ne $null){$QueryResults += $Results.value}  
  
            else{$QueryResults += $Results}  
  
            write-host "Method: $Method | URI $Uri | Found:" ($QueryResults).Count  
  
            $uri \= $Results.'@odata.nextlink'  
  
            }until ($uri \-eq $null)  
  
        }  
  
    Write-Progress \-Id 1 \-Activity "Executing query: $Uri" \-Completed  
  
    \# Return $QueryResults  
  
    $results |ConvertTo-Json \-Depth 5  
  
    }  
  
$resourceAppIdURI \= "https://graph.microsoft.com"  
  
$ClientID         \= "5ccaxxxx-xxxx-4xxx-8dxx-75xxxxx08833"   #AKA Application ID  
  
$TenantName       \= "contoso.onmicrosoft.com"             #Your Tenant Name  
  
$CredPrompt       \= "Auto"                                   #Auto, Always, Never,  RefreshSession  
  
$redirectUri      \= "https://RedirectURI.com"                #Your Application's  Redirect URI  
  
$Uri              \=  "https://graph.microsoft.com/v1.0/groups/d727xxxx-8x0x-47xx-88xx-3xxxxxxxxb95/drive/root:/example.xlsx:/workbook/worksheets('{5118xxxx-7xxE-43xx-Axxx-14xxxxxxFC13}')/Range(address='Sheet1!A1:G10')" #Query retrieve XLSX contents  
  
$Method           \= "Get"                                    #GET or PATCH  
  
$AccessToken      \= Get-AccessToken \-TenantName $TenantName \-ClientID $ClientID  \-redirectUri $redirectUri \-resourceAppIdURI $resourceAppIdURI \-CredPrompt  $CredPrompt  
  
$JSON \= @"  
  
 {  
  
 "userPrincipalName": "tom@contoso.com"  
  
 }  
  
"@ #JSON Syntax if you are performing a PATCH  
  
Invoke-MSGraphQuery \-AccessToken $AccessToken \-Uri $Uri \-Method $Method  

```