---
title: 'Over the Wall - with help from Azure'
date: 2018-07-22T22:01:00.003-07:00
draft: false
url: /2018/07/over-wall-with-help-from-azure.html
tags: 
- PowerShell
- VPN
- Azure
---

  

I had a long holiday in China recently. It had been a fun and eyes opening trip. Feels like the whole nation is in the middle of a “technology revolution”. Among all, Internet has become one of the core driving force. But while Wifi beomces a life essential here, the infamous Great Firewall (GFW) is also getting more effective. After tried numbers of paid and free VPN providers, I just could not find a reliable VPN service to simply let me post a picture on Facebook. I then decided it's time to bring the matter to my own hands. 

  

My VPN solution is pretty simple. It consist of a Squid proxy server and Azure P2S VPN. Access the Squid proxy server through its public IP alone will not help. The GFW will still be able to detect traffic destinations and block access successfully. A VPN tunnel ensure the traffic is encapsulated and prevents the GFW to block any traffic.

  

It is worth mentioning that this solution does incur cost! Azure resources like VPN Gateway have ongoing cost. I, fortunately have access to a subscription with $200 credit. You can check the price details from [here](https://azure.microsoft.com/en-gb/pricing/details/vpn-gateway/).

  

**Setup Squid Proxy**

OK, lets get started! 

So first, we need to build a Squid Proxy server in Azure. Below is the PowerShell script I use to provision the VM. It creates an Ubuntu Linux VM within a vNet. All resources are provisioned inside "tomlab-kr" resource group, which is located in Korea South region. The script also creates a Security Group which allows only SSH and TCP 7777 traffic. The custom TCP port will be used for Squid proxy service. 

  

\# Create a resource group in KoreaSouth

New-AzureRmResourceGroup  \-Name  "tomlab-kr"  \-Location  "Korea South"

  

\# Create a subnet configuration

$subnetConfig  \=  New-AzureRmVirtualNetworkSubnetConfig  \-Name  "proxysubnet1" \`  
\-AddressPrefix  192.168.188.0/24

  

\# Create a virtual network

$vnet  \=  New-AzureRmVirtualNetwork  \-ResourceGroupName  "tomlab-kr" \`  
\-Location  "Korea South"\-Name  "p2sVNET1"  \-AddressPrefix  192.168.0.0/16 \`  
\-Subnet  $subnetConfig

  

\# Create a public IP address and specify a DNS name

$pip  \=  New-AzureRmPublicIpAddress  \-ResourceGroupName  "tomlab-kr" \`  
\-Location  "Korea South"\-AllocationMethod  Static  \-IdleTimeoutInMinutes  4 \`

\-Name  "mypublicdns$(Get-Random)"

  

\# Create an inbound network security group rule for port 22

$nsgRuleSSH  \=  New-AzureRmNetworkSecurityRuleConfig  \-Name  "SGRuleSSH" \`  
\-Protocol  "Tcp"\-Direction  "Inbound"  \-Priority  1000  \-SourceAddressPrefix  \* \`

\-SourcePortRange  \*  \-DestinationAddressPrefix  \* \-DestinationPortRange  22 \`  
\-Access  "Allow"

  

\# Create an inbound network security group rule for port 7777

$nsgRuleWeb  \=  New-AzureRmNetworkSecurityRuleConfig  \-Name  "SGRuleWeb" \`  
\-Protocol  "Tcp" \-Direction  "Inbound"  \-Priority  1001  \-SourceAddressPrefix  \* \`

\-SourcePortRange  \*  \-DestinationAddressPrefix  \* \-DestinationPortRange  7777 \`  
\-Access  "Allow"

  

\# Create a network security group

$nsg  \=  New-AzureRmNetworkSecurityGroup  \-ResourceGroupName  "tomlab-kr" \`  
\-Location  "Korea South"\-Name  "LinuxSG"  \-SecurityRules  $nsgRuleSSH,$nsgRuleWeb

  

\# Define the credential for the VM

$cred  \=  Get-Credential  \-message  "login for the new vm"  \-UserName  "tom" 

  

\# Create a virtual network card and associate with public IP address and NSG

$nic  \=  New-AzureRmNetworkInterface  \-Name  "proxyVMNic" \`  
\-ResourceGroupName  "tomlab-kr"\-Location  "Korea South" \`  
\-SubnetId  $vnet.Subnets\[0\].Id \-PublicIpAddressId  $pip.Id \`  
\-NetworkSecurityGroupId  $nsg.Id

  

\# Create a virtual machine configuration

$vmConfig  \=  New-AzureRmVMConfig  \-VMName  "proxyvm1"  \-VMSize  "Standard\_DS1\_v2"  |  \`

Set-AzureRmVMOperatingSystem  \-Linux  \-ComputerName  "proxyvm1"  \-Credential  $cred  |  \`

Set-AzureRmVMSourceImage  \-PublisherName  "Canonical"  \-Offer  "UbuntuServer" \`  
\-Skus  "16.04-LTS"\-Version  "latest"  |Add-AzureRmVMNetworkInterface  \-Id  $nic.Id

  

\# Create the VM

New-AzureRmVM  \-ResourceGroupName  "tomlab-kr"  \-Location  "Korea South"  \-VM  $vmConfig

  

Once the VM is provisioned, the next step is to install and configure Squid on the Linux VM. 

The installation of Squid proxy server is pretty straight forward on a Ubuntu server. We simply run the command the below.

\# sudo apt-get install squid

To configure Squid proxy, we need to modify the squid configuration file. 

First, we make a backup copy of the configuration file, so in case we mess up, there is a clean copy to roll back to.

\# cp /etc/squid/squid.conf /etc/squid/squid.conf.bkcopy

Next, use your favorite text editor to modify the configuration file

\# sudo nano /etc/squid/squid.conf

Based on my tests, the default squid port 3128 is blocked by GFW. So the first thing to change is the default Squid port. In my case I changed it to 7777. 

http\_port 7777

Enable HTTP proxy access to all IPs.

http\_access allow all

Add ACL to allow VPN and local VM subnet IPs. Those lines should be added in two different sections in the configuration file. Please refer to the squid configuration guidance for details line locations.

**vpnsubnet** refers to the VPN Client Subnet, proxysubnet refers to the subnet where the Squid proxy server lives.

acl vpnsubnet 172.180.10.0/24

acl proxysubnet 192.168.188.0/24

  

http\_access allow proxysubnet

http\_access allow vpnsubnet

Now save the configuration file and restart squid service 

  

\# sudo systemctl restart squid

  

You can test the proxy by creating another VM in the same proxyvmsubnet. 

  

**Setup P2SVPN**

The steps to setup Azure Point-to-Site VPN is quite straight forward through Azure portal. Again, I will show you how this can be achieved through PowerShell script.

  

First, we need to create a Gateway Subnet within the vNet we created for the Squid proxy. The PowerShell script below simply created a subnet named "GatewaySubnet". This is a reserved name for the "Gateway Subnet".

  

$vNet  \=  Get-AzureRmVirtualNetwork  \-ResourceGroupName  "tomlab-kr"  \-Name  "p2sVNET1"

Add-AzureRmVirtualNetworkSubnetConfig  \-VirtualNetwork  $vNet  \-Name  "GatewaySubnet" \`  
\-AddressPrefix  "192.168.200.0/24"  
  

Set-AzureRmVirtualNetwork  \-VirtualNetwork  $vNet

  

  

Then we just need to provision a VPN Gateway within the subnet. 

Note: The last command "New-AzureRmVirtualNetworkGateway" may fail with unexpected errors. This is likely due to the Gateway Subnet actually takes a few hours to be properly configured in Azure. The solution is simply to try the command again after a few hours... 

  

\# Get gatewaysubnet

$subnet  \=  Get-AzureRmVirtualNetworkSubnetConfig  \-Name  "GatewaySubnet"  \-VirtualNetwork  $vNet  
  

$pip  \=  New-AzureRmPublicIpAddress  \-Name  "gwpip1"  \-ResourceGroupName  "tomlab-kr" \`  
\-Location  "korea south"  \-AllocationMethod  Dynamic

  
$ipconf  \=  New-AzureRmVirtualNetworkGatewayIpConfig  \-Name  "p2sgwIPconf"  \-Subnet  $subnet \`  
\-PublicIpAddress  $pip

  
\# Create VPN Gateway, remeber to wait for sunbnet ID to be generated  
\# (may take a few hours, use $subnet.id to check)

New-AzureRmVirtualNetworkGateway  \-Name  "p2sgw"  \-ResourceGroupName  "tomlab-kr" \`  
\-Location  "korea south"  \-IpConfigurations  $ipconf  \-GatewayType  VPN \`  
\-VpnType  RouteBased  \-GatewaySku  VpnGw1

  
After provision the VPN Gateway, the next step is to configure the **Client Address Space.** This is the IP range reserved for VPN clients. You may notice this is the IP range we defined in the Squid proxy configuration file.

  

\# configure the client address space with the VPN gateway

$Gateway  \=  Get-AzureRmVirtualNetworkGateway  \-ResourceGroupName  $RG  \-Name  $GWName

Set-AzureRmVirtualNetworkGateway  \-VirtualNetworkGateway  $Gateway \`  
\-VpnClientAddressPool  "172.180.10.0/24"

  

Next, we need to generate a pair of Root and Client certificates for the VPN Gateway authentication. 

  

\# Generate New Self Singed Certificate

$cert  \=  New-SelfSignedCertificate  \-Type  Custom  \-KeySpec  Signature  \`

\-Subject  "CN=P2SRootCert"  \-KeyExportPolicy  Exportable  \`

\-HashAlgorithm  sha256  \-KeyLength  2048  \`

\-CertStoreLocation  "Cert:CurrentUserMy"  \-KeyUsageProperty  Sign  \-KeyUsage  CertSign

  

\# Generate Client CCertificate

New-SelfSignedCertificate  \-Type  Custom  \-DnsName  P2SChildCert  \-KeySpec  Signature  \`

\-Subject  "CN=P2SChildCert"  \-KeyExportPolicy  Exportable  \`

\-HashAlgorithm  sha256  \-KeyLength  2048  \`

\-CertStoreLocation  "Cert:CurrentUserMy"  \`

\-Signer  $cert  \-TextExtension  @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")

  

The script generates both the Root and Client certificates under CurrentUserMy folder inside your certificate repo. 

Export the Root and Client certificates. The Root certificate needs to be exported without password as a **Base-64 encoded X.509 cer** file. The Client certificate needs to be exported with password in **PKCS #12 (.pfx)** format.

Run the PowerShell commands below to configure the VPN gateway with the Root certificate 

  

\# configure VPN Gateway with the root certificate

$P2SRootCertName  \=  "P2SRootCert.cer"

$filePathForCert  \=  "C:Downloads ootcert2.cer"

$cert  \=  new-object  System.Security.Cryptography.X509Certificates.X509Certificate2($filePathForCert)

$CertBase64\_3  \=  \[system.convert\]::ToBase64String($cert.RawData)

$p2srootcert  \=  New-AzureRmVpnClientRootCertificate  \-Name  $P2SRootCertName  \-PublicCertData  $CertBase64\_3

\# upload the public key information to Azure

Add-AzureRmVpnClientRootCertificate  \-VpnClientRootCertificateName  $P2SRootCertName  \`

  \-VirtualNetworkGatewayName  "p2sgw"  \-ResourceGroupName  "tomlab-kr"  \`

  \-PublicCertData  $CertBase64\_3

  

The solution is now ready for testing.  
  

**Connect from PC**

Setup P2S VPN client on PC is pretty easy. After download the VPN Client from Azure Portal, simply follow the prompts to install the client.

You may get a security warning from Windows 10. This can be simply ignored as you know this is a trusted app from Microsoft. Though it may look like the only option here is "**Don't run**"**,** as soon as you click **More Info,** you will be given the option to bypass the prompt.  

[![](https://4.bp.blogspot.com/-JrIGsAVrrJY/W1Veno2vCAI/AAAAAAAAKHI/0a1WiXKLGGsSKjkKVmkjJB0Ox7lMOPdWQCLcBGAs/s320/protectWarn1.png)](https://4.bp.blogspot.com/-JrIGsAVrrJY/W1Veno2vCAI/AAAAAAAAKHI/0a1WiXKLGGsSKjkKVmkjJB0Ox7lMOPdWQCLcBGAs/s1600/protectWarn1.png)

  

[![](https://1.bp.blogspot.com/-JCCZjkaJ0ps/W1Ve3HRoZ5I/AAAAAAAAKHM/VxJ6BgxW8hc86hPWHG7rs2sOf_-3x1nVACLcBGAs/s320/protectwarn2.png)](https://1.bp.blogspot.com/-JCCZjkaJ0ps/W1Ve3HRoZ5I/AAAAAAAAKHM/VxJ6BgxW8hc86hPWHG7rs2sOf_-3x1nVACLcBGAs/s1600/protectwarn2.png)

  

  

Once the client is installed, don't forget to change PC's proxy to the private IP of the Squid proxy server.

  

**Connect from iPhone**

By now you probably already jumped on Facebook posted 10 pictures and read 20 Emails from your Gmail inbox. But transferring photos from your phone to PC is still bit painful. You still miss the joy of sending those nice posts with a few clicks from your phone. OK, let's make that happen.

Unlike PC, there is no app to be installed on iOS. Instead, we need to install the client certificate we created for the VPN connection onto our iPhone. 

First, upload the .pfx file to OneDrive (get one, it's free and can be accessed in China). 

Next, install and open OneDrive app from your phone. Select the pfx file and choose Share. You should now see the "Copy Link" option. Click it to copy the link.

Open the URL with Safari (Not Chrome) from your phone. Safari will recognize it is a certificate and ask you if allow it to be installed. Click Allow to proceed. 

[![](https://3.bp.blogspot.com/-EcQJLjgAtls/W1Vfum4X2ZI/AAAAAAAAKHY/S0M6GEurlfgFu5kvvKkZjCipltbUwl_rACLcBGAs/s320/allowopen.png)](https://3.bp.blogspot.com/-EcQJLjgAtls/W1Vfum4X2ZI/AAAAAAAAKHY/S0M6GEurlfgFu5kvvKkZjCipltbUwl_rACLcBGAs/s1600/allowopen.png)

  

Follow the prompts to install.

  

[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[![](https://4.bp.blogspot.com/-bLSHMr58DFA/W1Vf6meBBEI/AAAAAAAAKHc/Nf10nNDnJYA28iQofaI_BC78oFr5hXp4ACLcBGAs/s320/installprofile.png)](https://4.bp.blogspot.com/-bLSHMr58DFA/W1Vf6meBBEI/AAAAAAAAKHc/Nf10nNDnJYA28iQofaI_BC78oFr5hXp4ACLcBGAs/s1600/installprofile.png)

  
  
  

[![](https://4.bp.blogspot.com/-roDmZm1ICUo/W1VgaQ6dPLI/AAAAAAAAKHw/nG_IhXa2dEE2Y_2sZD0YfMaJOxTawM35QCEwYBhgL/s320/pass.png)](https://4.bp.blogspot.com/-roDmZm1ICUo/W1VgaQ6dPLI/AAAAAAAAKHw/nG_IhXa2dEE2Y_2sZD0YfMaJOxTawM35QCEwYBhgL/s1600/pass.png)

  

After install the certificate, next step is to add a VPN configuration. 

The Server and Remote ID can be obtained from the XML file **vpnSettings.xml**downloaded alone with the VPN Client.

  

[![](https://3.bp.blogspot.com/-QIVWMkxonSQ/W1Vg7YzxONI/AAAAAAAAKH8/NOXE9z8eV3wgx_D0rhppLqE_vcOoOjYXwCLcBGAs/s320/vpnconfig.png)](https://3.bp.blogspot.com/-QIVWMkxonSQ/W1Vg7YzxONI/AAAAAAAAKH8/NOXE9z8eV3wgx_D0rhppLqE_vcOoOjYXwCLcBGAs/s1600/vpnconfig.png)

  

The Proxy settings within the VPN configuration somehow does not work. The only way is to add Proxy setting to local WiFi connection. 

  

[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[](https://www.blogger.com/blogger.g?blogID=3427010633633716171)[![](https://4.bp.blogspot.com/-cFOie6qWtJs/W1VhF9p7sHI/AAAAAAAAKIA/1DdLANwEnPQC53NhiQq3KkJRopAAGoY0wCLcBGAs/s320/wifiproxy.png)](https://4.bp.blogspot.com/-cFOie6qWtJs/W1VhF9p7sHI/AAAAAAAAKIA/1DdLANwEnPQC53NhiQq3KkJRopAAGoY0wCLcBGAs/s1600/wifiproxy.png)

  

After completed all these settings, fire up the VPN connection from your phone and you should now be able to access Facebook, Google, Instagram!