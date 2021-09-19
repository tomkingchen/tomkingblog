---
title: 'Install AWS CLI on WSL Ubuntu'
date: 2018-09-01T03:43:00.001-07:00
draft: false
url: /2018/09/install-aws-cli-on-wsl-ubuntu.html
---

Within Windows command prompt, AWS CLI does not provide autocomplete feature. According to AWS document, the feature is only available on “Unix-like systems”.

Luckily we have WSL (Windows Subsystem for Linux)on Windows 10. We can simply run AWS CLI from the WSL with autocomplete provided natively. Problem solved!

Here are the steps I took to get AWS CLI installed on my WSL Ubuntu.

Before we install AWS CLI package itself, we need to get Python package manager pip installed first.

Download pip install script. Notice I use –k here, this is because I am running this behind company proxy, the proxy changes HTTPS certificate to its own certificate. Without –k the command will fail. You can leave it out if you have direct Internet access

```
$ curl \-O \-k https://bootstrap.pypa.io/get-pip.py  

```

Next, the usual update apt command

```
$ sudo apt-gt update  

```

Then we download and install Python minimal

```
$ sudo apt install python-minimal  

```

Now we can install pip, the --trusted-host here is again due to the fact that I am downloading all these packages behind proxy.

```
$ python get-pip.py \--user \--trusted-host=files.pythonhosted.org  

```

Next, verify that pip is installed correctly.

```
$ pip \--version  

```

Finally, we can now use pip to install the AWS CLI.

```
$ pip install awscli \--upgrade \--user  

```

Verify the install by run aws --version.

[![image](https://lh3.googleusercontent.com/-3xgongnuYzw/W4ptPsakO2I/AAAAAAAAKQM/kl2cVBLQG4wDN5u81i71Ot6EB-2GSbv4QCHMYCw/image_thumb%255B5%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-chodvDmzCKs/W4ptOAkOEDI/AAAAAAAAKQI/3ONu1xB5Dyc9V2W9ZO0ML4_ST7bJgtOfgCHMYCw/s1600-h/image%255B7%255D)

Now let’s do something fun. Rather than enable the native AWS Completer, there is a pretty cool autocomplete tool call **aws-shell**. Here is its [GitHub link](https://github.com/awslabs/aws-shell).

To install aws-shell, run the pip install command below

```
$ pip install aws-shell –-user  

```

Run aws-shell to get into the shell. The tool will not only prompt for parameters, but can also retrieve information from AWS and promote for completion, like existing stack name, Security Group Ids, etc.

[![image](https://lh3.googleusercontent.com/-k9iwQjPluf4/W4ptR4tQImI/AAAAAAAAKQU/28exd4fNfacshoIazxd-8aAZaC9EGTlRgCHMYCw/image_thumb%255B9%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-di6gYNL1iZw/W4ptQkmIN6I/AAAAAAAAKQQ/Hw9ztyqITi4lK03w4-c4jJnk2GndoIGcACHMYCw/s1600-h/image%255B13%255D)

Have fun!