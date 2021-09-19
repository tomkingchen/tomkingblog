---
title: 'Test out PowerShell 7 new features in WSL1'
date: 2020-03-21T17:59:00.004-07:00
draft: false
url: /2020/03/test-out-powershell-7-new-features-in.html
---

Finally, PowerShell 7 is now GA! As a heavy WSL user, I was keen to see how some of its new features will work in WSL1 (Ubuntu 4.4.0-18362-Microsoft). Below are the tests I have done.  
  
**Installation in WSL**  
Download the binary from Github repo to a local folder /usr/share/powershell  

```
sudo wget https://github.com/PowerShell/PowerShell/releases/download/v7.0.0/powershell-7.0.0-linux-x64.tar.gz
```

  
Untar the file  

sudo tar xzvf powershell-7.0.0-linux-x64.tar.gz

  
Add path for your shell  

export PATH=/usr/share/PowerShell:$PATH

  
Reload .bashrc  

source .bashrc

  
Remove the tar ball  

sudo rm /usr/share/PowerShell/powershell-7.0.0-linux-x64.tar.gz

  
Run PowerShell 7 by run **pwsh**  
  
**Import Windows Modules in WSL**  
Install commonly Vendor released modules like VMware PowerCli  

Install-Module -Name VMware.PowerCli

  
Install .Net based modules also works fine.  

Install-Package PrtgApi

  
What about those module require Windows GUI, like Out-GridView? PowerShell 7 on Windows fully support this kind module. But for WSL some tweaks are required.  
  
First Install GraphicalTools  

Install-Module Microsoft.PowerShell.GraphicalTools

  

Tried to run Out-GridView but encountered Exception error.  

PS /home/tom>

Unhandled Exception: System.TypeInitializationException: The type initializer for 'OutGridView.Application.AvaloniaAppRunner' threw an exception. ---> System.Exception: XOpenDisplay failed

   at Avalonia.X11.AvaloniaX11Platform.Initialize(X11PlatformOptions options)

   at Avalonia.Controls.AppBuilderBase\`1.Setup()

   at OutGridView.Application.AvaloniaAppRunner.BuildAvaloniaApp() in D:\\a\\1\\s\\src\\OutGridView.Gui\\AvaloniaAppRunner.cs:line 34

   at OutGridView.Application.AvaloniaAppRunner..cctor() in D:\\a\\1\\s\\src\\OutGridView.Gui\\AvaloniaAppRunner.cs:line 31

   --- End of inner exception stack trace ---

   at OutGridView.Application.Program.Main(String\[\] args) in D:\\a\\1\\s\\src\\OutGridView.Gui\\Program.cs:line 26

  
To walk around that, we need to install XServer in Windows. I use VCSXRV downloaded from SourceForge.

  
The installation of VCSXRV is just click the next buttons.  
  
Keep VCSXRV running and back in WSL, exit pwsh type the command below  

export DISPLAY=localhost:0

  
That's it!. Let's test it out  

Get-Process | Out-GridView

  
Walah!!!  

[![](https://1.bp.blogspot.com/-rOkibpvbprM/Xmy37Gc92DI/AAAAAAAAKmo/xP_5EQPQvOsYMq5GqRkkJqPgl3V9ze0FwCLcBGAsYHQ/s400/image.png)](https://1.bp.blogspot.com/-rOkibpvbprM/Xmy37Gc92DI/AAAAAAAAKmo/xP_5EQPQvOsYMq5GqRkkJqPgl3V9ze0FwCLcBGAsYHQ/s1600/image.png)

  
  
  
  
  
  
  
  
  
  
  
  
  
**ForEach-Object Parallel option**  
This is another long wanted feature. I remember once I had to create thousands of large files on a hard disc and it was such a pain to wait for the loop to go through each creation process sequentially. With Parallel option, it can potentially save hours of waiting in this case.  
  
Interestingly though, it is not necessarily a silver bullet for shortening your loop's excution time. Let me show you some test results.  
  
**Parallel option slows the script**  

PS /home/tom/powershell/foo> measure-command {get-vm|foreach {$filename = $\_.Name; New-Item $filename;Set-Content .\\$filename "this is bunch of blah blah blah....." }}

  

Days              : 0

Hours             : 0

Minutes           : 0

Seconds           : 1

Milliseconds      : 16

Ticks             : 10161391

TotalDays         : 1.1760869212963E-05

TotalHours        : 0.000282260861111111

TotalMinutes      : 0.0169356516666667

TotalSeconds      : 1.0161391

TotalMilliseconds : 1016.1391

  

PS /home/tom/powershell/foo> measure-command {get-vm|foreach **\-parallel** {$filename = $\_.Name; New-Item $filename;Set-Content .\\$filename "this is bunch of blah blah blah....." }}

  

Days              : 0

Hours             : 0

Minutes           : 0

Seconds           : 12

Milliseconds      : 234

Ticks             : 122341726

TotalDays         : 0.000141599219907407

TotalHours        : 0.00339838127777778

TotalMinutes      : 0.203902876666667

TotalSeconds      : 12.2341726

TotalMilliseconds : 12234.1726

As you can see, in this case the Parallel option actually makes the whole script slower, a lot slower. Why is that? The new ForEach-Object -Parallel parameter set uses existing PowerShell APIs for running script blocks in parallel. These APIs have been around since PowerShell v2, but are cumbersome and difficult to use correctly. This new feature makes it much easier to run script blocks in parallel. But there is a fair amount of overhead involved.  
  
In our case, as the code calls to create a file is simple, putting the loop into Parallel adds a lot overhead and prolongs the running time of the script.  
  
**Parallel option improves performance**  
Now let's add a wait condition into the code.  

PS /home/tom/powershell/foo> measure-command {get-vm|ForEach-Object **\-parallel** {$filename = $\_.Name; New-Item $filename;Set-Content .\\$filename "this is bunch of blah blah blah.....";sleep 1}}

  

Days              : 0

Hours             : 0

Minutes           : 0

Seconds           : 35

Milliseconds      : 741

Ticks             : 357414908

TotalDays         : 0.000413674662037037

TotalHours        : 0.00992819188888889

TotalMinutes      : 0.595691513333333

TotalSeconds      : 35.7414908

TotalMilliseconds : 35741.4908

  

PS /home/tom/powershell/foo> measure-command {get-vm|ForEach-Object {$filename = $\_.Name; New-Item $filename;Set-Content .\\$filename "this is bunch of blah blah blah.....";sleep 1}}

  

Days              : 0

Hours             : 0

Minutes           : 2

Seconds           : 9

Milliseconds      : 848

Ticks             : 1298486786

TotalDays         : 0.00150287822453704

TotalHours        : 0.0360690773888889

TotalMinutes      : 2.16414464333333

TotalSeconds      : 129.8486786

TotalMilliseconds : 129848.6786

By adding sleep 1, each loop excution takes at least 1 second. This significantly increased the duration of the whole script. In this case the even with the overhead of Parallel, it is still beneficial to use the option.