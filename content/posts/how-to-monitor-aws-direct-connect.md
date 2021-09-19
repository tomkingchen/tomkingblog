---
title: 'Monitor AWS VPC Connectivity with Python'
date: 2019-12-12T13:14:00.000-08:00
draft: false
url: /2019/12/how-to-monitor-aws-direct-connect.html
---

We recently have the need to cutover our AWS Direct Connects to a different vendor. In order to carry out the change, I was tasked to find a way to monitor Direct Connect connectivities to our on premise network from our hundreds of VPCs in AWS.  
  
After some discussion with our network engineers and security team, the solution I end up using is to deploy a single EC2 instance into each those VPCs that has a connection to VGW. We then add those instance IPs into PRTG to monitor with Ping and Http sensors.  
  

[![](https://1.bp.blogspot.com/-abWMpg5NOS8/XfHLgZIlDMI/AAAAAAAAKkI/qHvZ2xtXPYUp_UsJv-kEi3Q77vVa1PuZgCLcBGAsYHQ/s320/DXMonitor.png)](https://1.bp.blogspot.com/-abWMpg5NOS8/XfHLgZIlDMI/AAAAAAAAKkI/qHvZ2xtXPYUp_UsJv-kEi3Q77vVa1PuZgCLcBGAsYHQ/s1600/DXMonitor.png)

To allow the instance to be deployed into the targeted AWS accounts, we use CloudFormation StackSet to push out a role into each of those accounts first. The role then allows the "Master Account" to have permission to create and update necessary resources within the target accounts.  
  
The instance uses a t3.nano tier alogn with Amazon hvm Linux2 AMI (Use our own hardened AMI in my case). A simple Nginx test page is installed on the instance to allow us to monitor TCP traffics. The total running cost of the solution depends on the number of instances/VPCs you have. For example, 100 instances cost around AU$500 a month.  
  
Yes, Lambda potentially can be cheaper, but it does not allow us to have real time monitoring as it does not support ICMP traffic. The underlaying container images for Lambda does not support that. [AWS recommended way](https://aws.amazon.com/blogs/networking-and-content-delivery/debugging-tool-for-network-connectivity-from-amazon-vpc/) to monitor VPC connectivities is to use Ping.  
  
The script can be easily transformed into a Lambda function and deployed with Serverless Framwork.  
  
The script can be found in my repo.  
[https://github.com/tomkingchen/DirectConnectMonitor](https://github.com/tomkingchen/DirectConnectMonitor)