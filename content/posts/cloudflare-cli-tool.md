---
title: "I wrote a Cloudflare CLI tool"
date: 2021-10-16T11:50:12+10:00
draft: false
---

I just wrote a Cloudflare CLI tool called `flare` 🔥! You can check it out from my GitHub repo [git@github.com:tomkingchen/cloudflare-cli.git](git@github.com:tomkingchen/cloudflare-cli.git).

The reason for creating the tool is mainly to help myself to quickly identify information hard to find through Cloudflare dashboard like Firewall rule ID. 

The current version is to focus on display information only rather than modifing configuration within Cloudflare. At work we use Terraform does most of the provisioning and reconfiguration of Cloudflare. This ensures proper change control and versioning. So there is no point to introduce another tooling for modifying configurations.

The tool is written in Python (3.8) with the use of [Click](https://click.palletsprojects.com) package. With `Click` I was able to easily create a AWS CLI style command line tool.

One of the use case of this tool is to get a list of all Firewall rules from Cloudflare and put them into a CSV file. I can then use this CSV as a lookup table in `Splunk` or other log analytic platforms. This enables us to live match WAF rule IDs obtained from `firewall logs` with rule description.

Here is the commands to generate the csv file.
```Bash
ZONES=$(flare list-zones |jq -r '.[].id')

for zone in $ZONES; do
  flare list-fw-rules --zoneid $zone |jq -r '.result[] | {rule_id: .id, rule_name: .description}'
done
```
Hope you find this tool somewhat useful. Feel free to drop in a PR if you have any awesome ideas want to add to it!