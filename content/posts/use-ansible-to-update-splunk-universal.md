---
title: 'Use Ansible to update Splunk Universal Forwarder Configuration'
date: 2021-02-20T20:55:00.002-08:00
draft: false
url: /2021/02/use-ansible-to-update-splunk-universal.html
---

Today we will look at how to use Ansible to update Splunk UF (Universal Forwarder) configuration. The benefits of using Ansible to achive this are:

\- Save the hassel to manually modify conf files of syslog-ng and splunk uf.

\- Codify Splunk UF configuratoin, so they can be version controlled via GitHub.

\- Automate multiple UFs update without the need to ssh to each single server. 

\- The playbook can also be used to configure newly provisioned Spunk UF.

Update log inputs on Splunk Universal Forwarder, normally invovles following tasks

1\. Modify two configuratoin files:

\- Syslog-ng conf file in conf.d/syslog-ng.conf. 

\- Syslog app inputs config file under Splunk UF installation folder.

2\. After the modification, Splunk service needs to be restarted.

For configuration files, we use Jinja2 template to simpilify its format. The Jinja2 templates use parameters supplied from CSV file to populiate the final conf file.

The Ansible playbook converts the template to conf file use template task. The conf file is saved in specified path on the remote server. Existing config files are overwritten if there are changes. 

Upon complete the config files update, Ansible use service task to issue a service restart of Splunk.

Code example can be found at my GitHub repo [https://github.com/tomkingchen/ansible-splunkuf-update](https://github.com/tomkingchen/ansible-splunkuf-update)