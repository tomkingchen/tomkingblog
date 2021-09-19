---
title: 'RDP to EC2 with SSM Port Forwarding'
date: 2019-10-19T14:05:00.001-07:00
draft: false
url: /2019/10/rdp-to-ec2-with-ssm-port-forwarding.html
---

Say you have a bunch of Windows servers hosted in AWS. The VPC they are in does not have VPN or Direct Connect connect back to your on premse network. Expose RDP port through public IP for these Windows servers is a very good way to get hacked. So how can we securely connect to the servers in this kind setup?  
  
Fortunately we have SSM for the rescue. In August, AWS announced a new feature for SSM Session Manager, which allows us to securely create tunnels between your EC2 instances deployed in private subnets and your local machine. You can read about the announcement [here](https://aws.amazon.com/blogs/aws/new-port-forwarding-using-aws-system-manager-sessions-manager/).  
  
Here are the steps you can setup for Windows Instances.  
1\. Configure the Windows EC2 as Managed Instances in SSM.  
This mainly involves assign a IAM EC2 Role to the instance with SSM policies. Since the focus of this post is about Session Manager Port Forwarding, I won't expand this too much. you can find more details about initial setup of SSM [here](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-profile.html).  
  
2\. For your existing Windows EC2s, you will need to update the SSM Agent on them to the latest version so it has the new feature. You can easily carry out SSM agent update via SSM State Manager.  
  
3\. In addition to AWS CLI or AWS PowerShell Toolkit, you will need to install Session Manager Plugin on your local machine. The plugin can be downloaded from [here](https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe).  
  
4\. Type the command below to initiate the Port Fowarding session. Make sure you have the correct region setup.  

```
Start-SSMSession -Target i-0x8x888xxxx8888xx -DocumentName AWS-StartPortForwardingSession -Parameter @{portNumber='3389';localPortNumber='9989'}  

```

The line above is pretty self-explanatory. You need to provide the Instance ID and the "Document" name, which is AWS term for SSM runbook. And also which port you want the fowarding.  
  
5\. Next, simply fire up Remote Desktop Connection (mstsc) and type **localhost:9989**.  
  
6\. That's it! You should now be able to RDP into the Windows EC2 instance **WITHOUT**:  

*   VPN/Direct Connection to the VPC
*   Assign public IP to the EC2 instnace
*   Open port 3389 in Security Group (That's right, you don't even need to do that!)

PS. Session Manager does allow Remote PowerShell access without any extra setup. However, the shell session is authenticated with a generic **ssm-user** account, which is not ideal for security audit.