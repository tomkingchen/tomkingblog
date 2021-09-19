---
title: 'Build a PDC in Azure with DSC'
date: 2018-09-07T17:19:00.000-07:00
draft: false
url: /2018/09/build-pdc-in-azure-with-dsc.html
---

There are a lot ARM templates out there can do this. But in this post, we will go through the nitty gritty of using DSC to automate the PDC setup. Before we begin, I assume you already know what DSC is and does. Otherwise, check it out [here](https://docs.microsoft.com/en-us/powershell/dsc/overview).

First, let’s build a new VM in Azure with these PowerShell commands. In this case, the VM will have direct Internet and can be accessed via Internet directly. Just to state here that this is definitely not the practice you want to adopt in production.

```
\# Create a new resource group  
New-AzureRmResourceGroup \-Name $rsgName \-Location $location  
  
\# Create a subnet configuration  
$subnetConfig \= New-AzureRmVirtualNetworkSubnetConfig \-Name "adsubnet" \-AddressPrefix 10.188.0.0/24  
  
\# Create a virtual network  
$vnet \= New-AzureRmVirtualNetwork \-ResourceGroupName $rsgName \-Location $location \-Name "tomlabVNET1" \-AddressPrefix 10.188.0.0/16 \-Subnet $subnetConfig  
  
\# Create a public IP address and specify a DNS name  
$pip \= New-AzureRmPublicIpAddress \-ResourceGroupName $rsgName \-Location $location \-AllocationMethod Static \-IdleTimeoutInMinutes 4 \-Name "tomlabpubdns$(Get-Random)"  
  
\# Create an inbound network security group rule for port 22  
$nsgRuleRDP \= New-AzureRmNetworkSecurityRuleConfig \-Name "tomlabSGRuleRDP"  \-Protocol "Tcp" \-Direction "Inbound" \-Priority 1000 \-SourceAddressPrefix \* \-SourcePortRange \* \-DestinationAddressPrefix \* \-DestinationPortRange 3389 \-Access "Allow"  
  
\# Create a network security group  
$nsg \= New-AzureRmNetworkSecurityGroup \-ResourceGroupName $rsgName \-Location $location \-Name "tomlabWinSG" \-SecurityRules $nsgRuleRDP  
  
\# Define a credential object  
$cred \= Get-Credential \-message "login for the new vm" \-UserName "tom"  
  
\# Create a virtual network card and associate with public IP address and NSG  
$nic \= New-AzureRmNetworkInterface \-Name "dc1VMNic" \-ResourceGroupName $rsgName \-Location $location \-SubnetId $vnet.Subnets\[0\].Id \-PublicIpAddressId $pip.Id \-NetworkSecurityGroupId $nsg.Id  
  
\# Create a virtual machine configuration  
$vmConfig \= New-AzureRmVMConfig \-VMName $vmName \-VMSize "Standard\_DS1\_v2" | \`  
Set-AzureRmVMOperatingSystem \-Windows \-ComputerName $vmName \-Credential $cred | \`  
Set-AzureRmVMSourceImage \-PublisherName "MicrosoftWindowsServer" \-Offer "WindowsServer" \-Skus "2016-Datacenter" \-Version "latest" | \`  
Add-AzureRmVMNetworkInterface \-Id $nic.Id  
  
\# Create the VM  
New-AzureRmVM \-ResourceGroupName $rsgName \-Location $location \-VM $vmConfig  

```

Next we need to create an Automation Account in Azure. I am sure you know how to get that done.

[![image](https://lh3.googleusercontent.com/-9op6zUhDfGk/W54DsEiGXBI/AAAAAAAAKSI/iKrn1VmNIWsE352QTnvB3_oRPEAcmdGjgCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-AUA-dVgFsQQ/W54DqxDorNI/AAAAAAAAKSE/2PEoWFEBHNQHkD5JeAicAj5TbiadY0w3gCHMYCw/s1600-h/image%255B2%255D)

As you are already in the portal, let’s get all the prerequisites ready before upload the DSC configuration.

First, go to **Modules** in the Automation Account and install following modules, if not already installed. They can be imported directly from **Gallery.**

*   xActiveDirectory
*   xStorage
*   xPendingReboot

[![image](https://lh3.googleusercontent.com/-1J6BOKQA3Ms/W54FFKD-mII/AAAAAAAAKSc/Xf4rR0jYp7Y8eG-ZcVOHMOQxQipgi2XogCHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-bscT94aYlLo/W54FD9M7RDI/AAAAAAAAKSY/A23GE_O5qa0uqD5kEFKEgnK7i8vqtkB0gCHMYCw/s1600-h/image%255B8%255D)

Next, go to **Credentials** in the Automation Account and create following two credentials. The reason behind this is well explained in this [Stackoverflow article](https://stackoverflow.com/questions/43508467/azure-automation-dsc-using-pscredential-in-dsc-configuration).

*   DCcred – Domain admin account
*   DCRecoveryCred – AD Recovery Password, this one you can put anything in the username, as it will not be used

Now we can start write DSC for the PDC. Below is the code. Save it as a ps1 file. You will notice it requires all those 3 modules we just installed in Azure Automation and the PSCredential it pulls from the Automation Account.

```
#Requires -module xActiveDirecotry  
#Requires -module xStorage  
#Requires -module xPendingReboot  
  
configuration DSCNewPDC               
{               
    \# Get Automation Credentials from Azure Automation Account  
    $safemodeAdministratorCred \= Get-AutomationPSCredential \-Name "DCRecoveryCred"  
    $domainCred \= Get-AutomationPSCredential \-Name "DCcred"  
              
    Import-DscResource \-ModuleName xActiveDirectory  
    Import-DscResource \-ModuleName xStorage  
    Import-DscResource \-ModuleName xPendingReboot  
              
    Node $AllNodes.Where{$\_.Role \-eq "Primary DC"}.Nodename               
    {                
        WindowsFeature ADDSInstall               
        {               
            Ensure \= "Present"               
            Name \= "AD-Domain-Services"               
        }              
              
        xWaitforDisk Disk2  
        {  
            DiskId \= 2  
            RetryIntervalSec \= 10  
            RetryCount \= 30  
        }  
  
        xDisk DiskF  
        {  
            DiskId \= 2  
            DriveLetter \= 'F'  
            DependsOn \= '\[xWaitforDisk\]Disk2'  
        }  
  
        xPendingReboot BeforeDC  
        {  
            Name \= 'BeforeDC'  
            SkipCcmClientSDK \= $true  
            DependsOn \= '\[WindowsFeature\]ADDSInstall','\[xDisk\]DiskF'  
        }  
  
        \# Optional GUI tools   
        WindowsFeature ADDSTools              
        {               
            Ensure \= "Present"               
            Name \= "RSAT-ADDS"               
        }              
              
        \# No slash at end of folder paths   
        xADDomain FirstDS               
        {               
            DomainName \= 'tomlab.briwave.com.au'               
            DomainAdministratorCredential \= $domainCred               
            SafemodeAdministratorPassword \= $safemodeAdministratorCred              
            DatabasePath \= 'F:\\NTDS'              
            LogPath \= 'F:\\NTDS'              
            DependsOn \= "\[WindowsFeature\]ADDSInstall","\[xDisk\]DiskF","\[xPendingReboot\]BeforeDC"      
        }              
              
    }               
}              

```

Apart from install the necessary roles and features, another thing worth noting is the configuration to check and use a 2nd disk for the NTDS log files. This is due to the factor that Azure OS disk by default has writing cache feature enabled, which could cause data corruption on AD DS. Here is the [MS document](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/identity/adds-extend-domain) supporting this argument.

Now, our original VM created with PowerShell does not have a 2nd disk attached. Let’s use the commands below to achieve that.

```
\# Add a managed disk to VM  
$rsgName \= "tomlab-au"  
$location \= "Australia Southeast"  
$vmName \= "testdc1"  
$diskName \= $vmName + '\_datadisk1'  
$diskConfig \= New-AzureRmDiskConfig \-AccountType PremiumLRS \-Location $location  \-CreateOption Empty \-DiskSizeGB 128  
$dataDisk1 \= New-AzureRmDisk \-DiskName $diskName \-Disk $diskConfig  \-ResourceGroupName $rsgName  
$vm \= Get-AzureRmVM \-Name $vmName \-ResourceGroupName $rsgName  
$vm \= Add-AzureRmVMDataDisk \-VM $vm \-Name $diskName \-CreateOption Attach  \-ManagedDiskId $dataDisk1.Id \-Lun 1  
Update-AzureRmVM \-VM $vm \-ResourceGroupName $rsgName  

```

Next we upload our DSC script to Azure.

[![image](https://lh3.googleusercontent.com/-PmWLiylbavQ/W54mrmDv6VI/AAAAAAAAKTI/va4Mf211H9cVQ2-p7Re08NxqzIIntbJLACHMYCw/image_thumb%255B3%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-DObUVrGLWE4/W54mqQBGA8I/AAAAAAAAKTE/EOfOuQsb-dAHUFttrOd3VPAuN8JlwYyVgCHMYCw/s1600-h/image%255B11%255D)

After upload the script, we need to compile it so it can be pushed to the Azure Windows VM. This is done through the PowerShell script below. As you can see, we need to specify the parameters and configData.

```
Param(  
\[Parameter(Mandatory=$True)\]  
\[String\]$rsgName,  
\[Parameter(Mandatory=$True)\]  
\[String\]$autoAccountName,  
\[Parameter(Mandatory=$True)\]  
\[String\]$configName  
)  
$ConfigData \= @{  
       AllNodes \= @(  
              @{  
                     NodeName \= "\*"  
                     PSDscAllowPlainTextPassword \= $True  
              },  
              @{  
                     NodeName \= "testdc1"  
            Role \= "Primary DC"  
              }  
       )  
}  
\# Script to complie DSC Configuration in Azure Automation Account  
Start-AzureRmAutomationDscCompilationJob \-ResourceGroupName $rsgName  \-AutomationAccountName $autoAccountName \-ConfigurationName $configName  \-ConfigurationData $ConfigData  

```

If all goes well, you should see something like below.

[![image](https://lh3.googleusercontent.com/-Dsm52ZpZs24/W54mtK9zyII/AAAAAAAAKTQ/NXPyG2FN3B8frVBx_EstrNcy5PJwAd3rACHMYCw/image_thumb%255B4%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-Qzt6fP_2SNA/W54msVTaY_I/AAAAAAAAKTM/fhTv40ScJVUYmEZf2dA8fTffZxMyLUqsQCHMYCw/s1600-h/image%255B14%255D)

Next, we need to apply the complied configuration to the node (Windows VM). You will need to reboot the server after the initial DSC push. This is to apply those roles and features.

Once the configuration is applied, it will take a while for Azure Automation to get the final status.

[![image](https://lh3.googleusercontent.com/-dvHNKacGWHU/W54mvU351WI/AAAAAAAAKTY/rhejI5gntbcXeOJW9uG9XwWlNVgdkisxACHMYCw/image_thumb%255B6%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-iWBT3FWKZhM/W54muZZBQEI/AAAAAAAAKTU/CfsX2SLF2c4-YGyDafMl7jl4e96fW532gCHMYCw/s1600-h/image%255B20%255D)