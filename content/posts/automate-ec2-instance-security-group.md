---
title: 'Automate EC2 Instance Security Group Rules Update'
date: 2021-06-04T22:47:00.000-07:00
draft: false
url: /2021/06/automate-ec2-instance-security-group.html
---

Ever come into the situation where you need to whitelist a long list of IPs for a EC2 instance? It can be painful to add them manually one by one. On top of that, what if these IPs change on a regular basis? You are in luck! I will show you how to update Security Group rules automatically using Python.

Here's my use case. I got an EC2 instance takes syslog feeds from VMWare's WorkspaceOne. For security sake, it should only allow IPs owned by VMware SaaS solution, which are all listed in this [KB](https://kb.vmware.com/s/article/2960995). There are around 990 IPs and subnets need to be added. Plus, they are constantly get updated.

The solution I put in place is to scrape the IPs off the KB webpage, and then add them into security groups attached to the EC2 instance. Sounds like a piece of cake, right? Well, there are a few huddles we need to address first.

First, the VMware KB page is not a static HTML page. The IP list is rendered from Javascript. If we just use Requests library, it will return None as the page is not rendered. To overcome that, I use Selenium's ChromeDriver to mimic a browser request and get the rendered HTML content. The code below shows how this is achieved.

options = Options()

options.headless = True

driver = webdriver.Chrome('/usr/bin/chromedriver', options=options)

driver.get('https://kb.vmware.com/s/article/2960995')

time.sleep(5)

page = driver.page\_source 

The next step is to retrieve the IP list from the HTML page using BeautifulSoup.

soup = BeautifulSoup(page, 'html.parser')

page\_content = soup.find(id='article\_content')

page\_containers = page\_content.find\_all('div', class\_='container')

ip\_list\_content = page\_containers\[1\].find('div', class\_ = 'content')

There is a soft limit of 5 Security Groups per network interface in AWS. Each Security Group by default can contain no more than 60 ingress rules. With over 900 IPs in the VMware KB page, we need to reach out to AWS Support uplift those software limits first. I end up raise the limit to 10 x SGs per interface and each SG can contain up to 100 ingress rules.

Once this is in place, we can then create the SGs based on the IP list and attach the SGs to the instance. Existing SGs will be detacched and removed prior the new SG attachment.

Here is the completed code [https://github.com/tomkingchen/auto-sg-update](https://github.com/tomkingchen/auto-sg-update).