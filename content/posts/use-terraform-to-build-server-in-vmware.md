---
title: 'Use Terraform to build server in VMware'
date: 2019-05-18T00:25:00.001-07:00
draft: false
url: /2019/05/use-terraform-to-build-server-in-vmware.html
---

Like Cloud Formation and ARM Templates, Terraform enables the way of Infrastructure as Code to provision resources in Clouds, but it also works with on premise infrastructures like VMware vSphere and NSX. I recently have been working on the automation of on premise server provision process. The goal is to provision a Ubuntu server on our vSphere 6.5 environment with iPerf3 installed and configured.  
  
It surprises me that there aren’t many useful resources/examples out there when comes to using Terraform with VMware. Yes, there are tons of blog posts about how to build a VMware VM with Terraform, but almost all of those are just touched on very very basic stuff. I can’t find any good reference for how to install and configuration applications within the VM. Without those, I seriously doubt the value of using IaaC. A simple VMware Template with ClickOps will be way more efficient.  
  
I did end up finding a very detailed repo about using Terraform to build a Kubernate cluster in VMware in GitHub. But this only make me feel more frustrated. This picture reflects exactly how I feel after went through that repo…  

[![](https://i.kym-cdn.com/photos/images/original/000/572/078/d6d.jpg)](https://i.kym-cdn.com/photos/images/original/000/572/078/d6d.jpg)

  
  
Another surprise finding is the lacking of native means to securely protect sensitive data like secret keys and passwords alike. To connect to the vCenter, I have the choice of either save my admin password inside the variables file or type the password unmasked in command prompt. Credentials for VMs can easily be leaked through state file. After consulted with one of our seasoned Terraform users and some Googling, it seems the best way to securely use Terraform is take advantage of some of the 3rd party services, e.g. store passwords in Secret Manager, share state file through S3 buckets.  
  
In my case, all resources are on premise. I don’t want to call AWS APIs just so I can enter my password for vCenter. I decided to simply use PowerShell script to get the credential then save it into variables file during the runtime of the Terraform template. Once the template is applied, the PowerShell script will reverse the variables file to its original version, which cleans out the plaintext password. One thing I should but haven’t done is to add to the script is to clean out memory.  
  
To avoid store plain password for the VM in the template, I configure the VM template to use SSH key pair for authentication. This is not difficult, but as a ClickOps Windows guy, it does take me sometime to figure out. WSL again proves its value in this case. The template itself is fairly simple. I got most references from Terraform’s official document.  
  
The whole script + template example can be found in my GitHub repo [here](https://github.com/tomkingchen/terraform-linux-vmware.git). As usual, hope you find it helpful.