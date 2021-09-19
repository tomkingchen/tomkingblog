---
title: 'Infrastructure as Code with CloudFormation'
date: 2019-01-04T22:37:00.000-08:00
draft: false
url: /2019/01/infrastructure-as-code-with.html
---

Recently I was working on a server migration task, which is to move a Windows IIS web server to AWS. The server’s sole purpose is to redirect bunch of the short URLs to some of the most frequently used long URLs. E.g. if user type in “o365/“ in browser, it will be redirected to [https://portal.office365.com](https://portal.office365.com).

Instead of uplifting the whole Windows server to AWS, I have decided to use a Linux server with Apache to replace this box. Other options like ALB, S3 HTTP Redirection and Route 53 were considered. But none of them can redirect short hostname like “o365”. I have also thought about the idea of using container and serverless options, but given all we need is a single redirect service and the a fixed IP is needed for alias lookup, they are not suitable in this case.

The Linux server is provisioned through a CloudFormation template. This allows automation of the whole deploy process. Furthermore, any future updates to the Apache service can also be carried out with an update of the CloudFormation stack.

Let’s start with the parameters of the template. In the parameters, we provide VPCId, serverName, SubnetId, KeyPairName, S3URL and an UpdateString. A lot of these are self explanatory. I have provide reasons for why we need UpdateString in the later part of this article.

```
Parameters:   
  VPCId:  
    Type: String  
    Description: VPC ID  
  ServerName:  
    Type: String  
    Description: Server name  
  VPCSubnetId:  
    Type: String  
    Description: Subnet ID in the format of subnet-c1xxxxxx  
  KeyPairName:  
    Type: String  
    Description: Key Pair name for the instance  
  S3URL:  
    Type: String  
    Description: Pre-Signed S3 URL for httpd.conf  
  UpdateString:  
    Type: String  
    Description: Update String to trigger cfn-hup  
Mappings:   
  RegionMap:  
    ap-southeast-2:  
      ALinux : 'ami-00c3d41691e25e54c'  

```

The CloudFormation template contains only two resources: a Security Group and EC2 Instance.

There is nothing special about the security group. It simply allows HTTP and SSH access to the box.

```
  redirSG:  
    Type: AWS::EC2::SecurityGroup  
    Properties:  
      GroupName: sec-httpredir  
      GroupDescription: Security Group for URL redirect  
      VpcId:   
        Ref: VPCId  
      SecurityGroupIngress:  
      \- IpProtocol: tcp  
        FromPort: 80  
        ToPort: 80  
        CidrIp: 10.0.0.0/8  
      \- IpProtocol: tcp  
        FromPort: 22  
        ToPort: 22  
        CidrIp: 10.0.0.0/8  
      Tags:  
        \- Key: Name  
          Value: sec-httpredir    

```

The EC2 resource though, needs quite bit work to for Apache configuration. It uses cfn-init to install and configure Apache. And here is how it is done.

First, we define the AWS::CloudFormation::Init metadata. Note, all this key words are case sensitive. So be mindful when you type them in. Once I spend hours to workout it is a lower case caused by template to fail.

```
  Metadata:  
      AWS::CloudFormation::Init:  
        config:  
          packages:  
            yum:  
              httpd: \[\]  
          files:  
            "/etc/cfn/cfn-hup.conf":  
              content: !Sub |  
                \[main\]  
                stack=${AWS::StackId}  
                region=${AWS::Region}  
                verbose=true  
                interval=5  
              mode: "000400"  
              owner: root  
              group: root  
            "/etc/cfn/hooks.d/cfn-auto-reloader.conf":  
              content: !Sub |  
                \[cfn-auto-reloader-hook\]  
                triggers=post.update  
                path=Resources.WebServer.Metadata.AWS::CloudFormation::Init  
                action=/opt/aws/bin/cfn-init --verbose --stack ${AWS::StackName} --resource WebServer --region ${AWS::Region}  
                runas=root  
              mode: "000400"  
              owner: root  
              group: root  
            "/etc/httpd/conf/httpd.conf":  
              source: !Ref S3URL  
              mode: "000400"  
              owner: "root"  
              group: "root"  
            "/var/www/html/index.html":  
              content: !Sub |  
                <html>  
                <body>  
                <h1>${UpdateString}</h1>  
                </body>  
                </html>  
              mode: "000644"  
              owner: "apache"  
              group: "apache"  
          services:  
            sysvinit:  
              cfn-hup:  
                enabled: "true"  
                ensureRunning: "true"  
                files:  
                  \- "/etc/cfn/cfn-hup.conf"  
                  \- "/etc/cfn/hooks.d/cfn-auto-reloader.conf"  
              httpd:  
                enabled: "true"  
                ensureRunning: "true"  
          commands:  
            replacePrivateIP:  
              cwd: "/etc/httpd/conf"  
              command: sed -i s@127.0.0.1@$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)@g httpd.conf  
            restartApache:  
              cwd: "/etc/httpd"  
              command: service httpd restart  

```

With cfn-init, we tell the instance to configure cfn-hup service, which allows the instance to be updated when the CloudFormation Stack is updated.

Then we tell the instance to load Apache configure file httpd.conf from a S3 bucket. The S3URL parameter is a pre-signed S3 URL, which is only validate for 1 hours by default. This is to ensure the security of the configuration file, although it contains anything sensitive. Still, it’s better to be secure than just leave the gate open.

Next, cfn-init will load a default homepage indiex.html with the given code, which displays the UpdateString. This parameter can be anything, it simply tricks CloudFormation service to update the stack because something has changed. Otherwise, even if you made a change in the httpd.conf file. CloudFormation won’t be able to detect the change and will not initiate cfn-hup to rerun the configuration procedure.

In the services section, the template tells cfn-init to keep cfn-hup and Apache service to keep running.

In the command section, the template initiates two commands.

\- one use powerful sed command to replace 127.0.0.1 local IP with the EC2 instance’s private IP address in the Apache config file. This is to ensure Apache service is always configured with the correct private IP of the instance.

\- The next command restarts Apache service after each configuration update.

That summarizes the cfn-init configuration. Now let’s look at the EC2 properties section.

The Properties contains the usual elements: KeyName, ImageId, InstanceType, NetworkInstances and UserData.

```
Properties:  
      KeyName: !Ref KeyPairName  
      ImageId:   
        Fn::FindInMap:  
          \- RegionMap  
          \- Ref: 'AWS::Region'  
          \- ALinux  
      InstanceType: t2.micro  
      NetworkInterfaces:  
        \- AssociatePublicIpAddress: "false"  
          DeviceIndex: "0"  
          GroupSet:  
            \- !Ref redirSG  
          SubnetId: !Ref VPCSubnetId  
      UserData:   
        "Fn::Base64":  
          !Sub |  
            #!/bin/bash -xe  
            yum update -y aws-cfn-bootstrap  
            /opt/aws/bin/cfn-init -v -s ${AWS::StackName} -r WebServer --region ${AWS::Region} || error\_exit 'Failed to run cfn-init'  
            yum -y update  
            /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource WebServer --region ${AWS::Region}  
      Tags:  
        \- Key: Name  
          Value: !Ref ServerName  
        \- Key: owner  
          Value: "SysEng"  
        \- Key: ChangeTag  
          Value: !Ref UpdateString  
    CreationPolicy:  
      ResourceSignal:  
        Count: 1  
        Timeout: "PT10M"  

```

UserData is where we specify cfn-init to configure which stack and which resource.

cfn-signal provides the result of cfn-init. If anything go wrong with cfn-init, cfn-signal will provide the result to “ResourceSignal” defined in “CreationPolicy”. The CreationPolicy defines if within 10 mins the stack didn’t return any result, it will consider the resource creation failed and cause the stack creation/update to roll back.

The CloudFormation template code example can be downloaded from my GitHub repo here.