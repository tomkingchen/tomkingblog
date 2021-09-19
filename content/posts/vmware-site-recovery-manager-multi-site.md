---
title: 'VMware Site Recovery Manager Multi-Site Pair Deployment'
date: 2019-09-06T23:22:00.003-07:00
draft: false
url: /2019/09/vmware-site-recovery-manager-multi-site.html
---

I was recently involved in a data center migration project, which used VMware SRM (Site Recovery Manager) as the migration tool to move virtual machines between 3 DCs. The diagram below shows how the setup looks like. The version of SRM is 8.1.  
  
\[SiteA\] <----> \[SiteB\] <----> \[SiteC\]  
  
VMware documentation refer the above scenario as Shared Recovery Site. For each site-pair, you will need to deploy individual SRM server to ensure the **SRM Plug-in ID** is unique to that pair. So in our case, SiteB-C pair requires a new SRM server in Site B and Site C.  
  
**Install SRM**  
  
The steps below shows the configuration details of the installation process on each SRM server.  
1\. Provide the PSC server details of the site  
2\. Provide the vCenter server details of the site  
3\. Provide the local SRM extension details  
4\. This is the most important part of the configuration. Make sure you choose **Custom Site Recovery Manager Plug-in Identifier**. The Plug-in ID needs to be **the same** on both Site B and Site C SRM instance.  
5\. Choose certificate  
6\. Configure Embedded Database configuration. Make sure your Windows Firewall allows the database port, as by default it is blocked even for localhost!  
7\. Choose service account. My recommendation is to use Local System account, so you don't have to manage another privileged AD account!  
  
**Configure SRM Site Pair**  
1\. Log into SRM portal. Normally its https://srmservername/dr  
2\. Create New Site Pair  
3\. Select local vCenter server  
4\. Select the remote site vCenter server you want to pair  
5\. For replication method, we will be using Storage Replication. This require us to install correct Storage Replication Adapter on the SRM server (In our case it's HPE Nimble). Usually the SRA can be downloaded from VMware site.  
6\. For the site pair, check SRA is in OK status  
  
**Configure Nimble Array for Storage Replication**  
Create a new Replication Partner in Nimble  
Choose on-premise Replicatin Partner  
Put in the remote array details. Make sure you type the correct Array Group Name with the right case. It is case sensitive for Nimble (Given the controller basically is a Linux server).  
Repeat the same step on remote Replicatin Partner.  
Create Volume Collection with the configured Replication Partner.