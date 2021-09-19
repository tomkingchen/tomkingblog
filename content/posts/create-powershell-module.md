---
title: 'Create a PowerShell Module'
date: 2018-09-27T02:18:00.000-07:00
draft: false
url: /2018/09/create-powershell-module.html
---

Recently I came across an issue on our Hyper-V Cluster. One of the VM was stuck in the “Stopping” state. I had to force the VM to shutdown by kill its process on the Hyper-V host. To do so, I first find out the VM’s GUID and then kill the process with the same GUID. Needless to say, the whole process can be achieved with the PowerShell commands below.

```
\# Get the VM GUID and find the process with the GUID  
$VM \= Get-VM \-Name $VMName \-ErrorAction Stop  
$VMGUID \= $VM.Id  
$VMWMProc \= (Get-WmiObject Win32\_Process | Where-Object {$\_.Name \-match 'VMWP' \-and $\_.CommandLine \-match $VMGUID})  
      
\# Kill the VM process  
Stop-Process ($VMWMProc.ProcessId) –Force  

```

This is a pretty short and simple script, which is perfect for making a module. We have number of Hyper-V hosts across our environment. Instead of copying the script around, module will help us better organize the code and make it more re-usable. I also like to share this small function with our global team, which module will serve nicely for this purpose.

Alright, let’s convert the script into a module then. Here’s the script Kill-VM.ps1. Let’s save it as Kill-VM.psm1. And now we have a module! That’s it?! Noooo… of course not. There are a few things we need to change with the script, before we can call it a proper module.

```
#requires -module Hyper-V  
#requires -runasadministrator  
  
\[CmdletBinding(SupportsShouldProcess=$True)\]  
Param(  
    \[Parameter(Mandatory\=$true,  
        HelpMessage\="The VM Name"  
        )\]  
    \[String\]$VMName  
)  
  
Try{  
    #Get a VM object from local HyperV host  
    $VM \= Get-VM \-Name $VMName \-ErrorAction Stop  
    $VMGUID \= $VM.Id  
    $VMWMProc \= (Get-WmiObject Win32\_Process | Where-Object {$\_.Name \-match 'VMWP' \-and $\_.CommandLine \-match $VMGUID})  
      
    \# Kill the VM process  
    Stop-Process ($VMWMProc.ProcessId) –Force  
    Write-Output "$VMName is stopped successfully"  
}Catch{  
    $ErrorMsg \= $Error\[0\] #Get the latest Error  
    Write-Output $ErrorMsg  
}  

```

  

First, we need to give a name to our module. Let’s call it “VMKiller”! “VMTerminator” is cooler, but a bit too long for my like…

Create a folder named “VMKiller” under “C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\Modules”. There are a few other default folders for PowerShell modules. You can find them out by run the command below.

```
$env:PSModulePath  

```

Save our script as VMKiller.psm1 into the “VMKiller” folder.

In the psm1 file, wrap the parameter and try sections into a function call “Kill-VM”.

We will also add Begin, Process and End section to handle Pipeline input for the function. This is not essential in this particular case. But will be useful, if we have ForEach loop in the function. The Process section will help us properly handle multiple objects inputted from pipeline.

Below is the code in VMKiller.psm1 after above changes.

```
Function Kill-VM  
{  
    \[CmdletBinding(SupportsShouldProcess=$True)\]  
    Param(  
        \[Parameter(Mandatory\=$true,  
            ValueFromPipeline\=$True,  
            ValueFromPipelineByPropertyName\=$True,  
            HelpMessage\="The VM Name"  
            )\]  
        \[String\]$VMName  
    )  
    Begin{}  
    Process{  
        Try{  
            #Get a VM object from local HyperV host  
            $VM \= Get-VM \-Name $VMName \-ErrorAction Stop  
            $VMGUID \= $VM.Id  
            $VMWMProc \= (Get-WmiObject Win32\_Process | Where-Object {$\_.Name \-match 'VMWP' \-and $\_.CommandLine \-match $VMGUID})  
              
            \# Kill the VM process  
            Stop-Process ($VMWMProc.ProcessId) –Force  
            Write-Verbose "$VMName is stopped successfully"  
        }Catch{  
            $ErrorMsg \= $\_.Exception.Message #Get the latest Error  
            Write-Verbose $ErrorMsg  
        }  
    }  
    End{}  
}  

```

Next, to make our module more understandable to future readers, we will add some Comment based Help information into the code. Comment based Help will allow users to read about the module function by using Get-Help command. You can read more about Comment Based Help from [here](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comment_based_help?view=powershell-6).

```
<#  
.SYNOPSIS  
Kill a VM Process.  
  
.DESCRIPTION  
Use this module when a VM is stuck at Stopping state. It will kill the VM Process on the HyperV host.  
  
.PARAMETER VMName  
Name of the virtual machine.  
  
.EXAMPLE  
C:\\PS>Kill-VM -VMName "contoso-av1"  
  
#>  

```

The VMKiller module so far only contains a single function, and a single psm1 file. With what we have done so far, it is ready to be used. However, there are some additional bits I would like to show you. 

It is not necessary to create Module Manifest, while we have only one single psm1 file. Without the Manifest, PowerShell will try to load _ModuleName.DLL_ first. If that is not successful, it will then try _ModuleName.psm1_. But in this case we will create one anyway. To create the manifest, use the command below.

```
New-ModuleManifest –path .\\VMKiller.psd1  –RootModule VMKiller.psm1  

```

With the module file, you only need to modify one line, on line 72, remove the wildcard ‘\*’ and replace it with ‘Kill-VM’ function. This is to specifically tell PowerShell which function to be exposed from this module. When you have a lot of functions in a module, this will improve the module loading performance.

[![image](https://lh3.googleusercontent.com/-XAaCd9_M5xI/W68ZoDYoVYI/AAAAAAAAKUM/hltE6FXqc04VR9SJyh9euEJfSaJ8BS8xwCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-iiiXyEYd0Do/W68ZnCFfrVI/AAAAAAAAKUI/xhO2CT1hSigSdltfiC8uIbOt6nFB5eXWQCHMYCw/s1600-h/image%255B2%255D)

Now save all the files, let’s try load the VMKiller module and put it to use.

First, let’s load the module with Import-Module. You will get a warning about unapproved verb. This is because “Kill” is not something Microsoft will use in their command… They need to make sure everything is 200% politically correct. You can just ignore that, as I do not care.

[![image](https://lh3.googleusercontent.com/-87bwxrNUBZ8/W68Zp2Zp3TI/AAAAAAAAKUU/gcpG7nY0UuYfCZnRj33q2_6qdrE_I4LawCHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-pAjcLBNT_Dg/W68ZpO6D9UI/AAAAAAAAKUQ/1KOXv84aPU8wGNfBpdj-UNMFXHnRoxdOACHMYCw/s1600-h/image%255B8%255D)

Next, let’s see what users will see if they try to learn about the function with Get-Help.

[![image](https://lh3.googleusercontent.com/-USLv9gbtAQI/W68ZsZSlb0I/AAAAAAAAKUc/dmZc8JWkLsY7bfT8zKpSs08i_qCnDBjIQCHMYCw/image_thumb%255B1%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-ChUA8LnIYxs/W68ZrW4M2fI/AAAAAAAAKUY/sSBC_mA7eGgZhZAvCalFbVPvc7cB5dBNQCHMYCw/s1600-h/image%255B5%255D)

Now they know how to use the cmdlet, let’s put it to real use. Let’s stop a running VM.

[![image](https://lh3.googleusercontent.com/-cFMtMnrHejc/W68ZuLk6pXI/AAAAAAAAKUk/CnhCfcBNZPwz1q01mtGJER1fx75KfIHhgCHMYCw/image_thumb%255B3%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-WbVWPw30dMc/W68ZtHMbVUI/AAAAAAAAKUg/wX1egnCDmGYcg0SQ4bzPDHJmKxAM2ZWewCHMYCw/s1600-h/image%255B11%255D)

And that’s it! You now have a module can be deployed to any other Hyper-V hosts. You can even use DSC to make sure it is installed on all Hyper-V hosts!