---
title: 'Setup Cross Account S3 Access for Cloudberry Drive'
date: 2019-04-13T16:05:00.000-07:00
draft: false
url: /2019/04/setup-cross-account-s3-access-for.html
---

I recently run into a scenario, which one of EC2 instances in our production AWS account (IT) need to access a S3 bucket hosted in a separate account (Marketing). The EC2 instance is a Windows 2008 R2 server. It runs Cloudberry Drive to map the S3 bucket as a local volume for a local application to retrieve the data off it.  
  
The easiest way to make this work is to create an IAM user in the and assign it with Access keys. But this is against [AWS IAM best practice](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html#delegate-using-roles). Cloudberry Drive does provide the option to use Role for S3 bucket access. Though their documentation is a bit lacking on how to setup this in a cross account scenario. After some Googling, it turns out to be fairly straight forward. Here's how I did it.  
  
Let's start with a picture as it helps to clarify where things are in this two account setup.  

[![](https://2.bp.blogspot.com/-KDI-f-Y1UlQ/XLJyRHj_VgI/AAAAAAAAKgM/6iN6kIxAD3MvS8_GntWo5VxJpRlTgGzaQCLcBGAs/s640/diagram.png)](https://2.bp.blogspot.com/-KDI-f-Y1UlQ/XLJyRHj_VgI/AAAAAAAAKgM/6iN6kIxAD3MvS8_GntWo5VxJpRlTgGzaQCLcBGAs/s1600/diagram.png)

  
First, in the **IT account (111111111111)**:  
1\. Create an IAM role **market-s3-role** with the following policy: **fullaccess-marketing-bucket**. The policy allows access to the S3 bucket **market-s3-bucket** and its objects in the Marketing account (88888888888).  
Here’s the policy JSON file.  
  

```
{  
    "Version": "2012-10-17",  
    "Statement": \[{  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "arn:aws:iam::111111111111:role/market-s3-role"  
            },  
            "Action": \[  
                "s3:ListBucket",  
                "s3:GetBucketLocation"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::market-s3-bucket"  
            \]},  
        {  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "arn:aws:iam::1111111111:role/market-s3-role"  
            },  
            "Action": \[  
                "s3:GetObject",  
                "s3:PutObject",  
                "s3:DeleteObject"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::market-s3-bucket/\*"  
            \]  
        }  
    \]  
}  

```

  
2\. Attach the role to the EC2 instance (the Windows server). This can be done when the instance is online.  

[![](https://1.bp.blogspot.com/-NuzVO1I9tew/XLJy9-c9-VI/AAAAAAAAKgU/WAmtv-cFHqcZ1FXxf35k7hRb36WjU_b6gCLcBGAs/s640/attachec2role.png)](https://1.bp.blogspot.com/-NuzVO1I9tew/XLJy9-c9-VI/AAAAAAAAKgU/WAmtv-cFHqcZ1FXxf35k7hRb36WjU_b6gCLcBGAs/s1600/attachec2role.png)

  
  
3\. In CloudBerry Drive options, set the Storage Account to **Use AWS IAM role policy**.  

[![](https://2.bp.blogspot.com/-kobupxQ026M/XLJzaN1372I/AAAAAAAAKgg/zKNZGd5-UqgDkiLgmk5GS2VqE-un5RtrACLcBGAs/s400/driveAccountsetup.png)](https://2.bp.blogspot.com/-kobupxQ026M/XLJzaN1372I/AAAAAAAAKgg/zKNZGd5-UqgDkiLgmk5GS2VqE-un5RtrACLcBGAs/s1600/driveAccountsetup.png)

  
  
Now, switch to the **Marketing account (888888888888)**:  
1\. Create a bucket policy to the S3 bucket **market-s3-bucket** as below. As you can see, it’s basically the same as the Role Policy in the IT account.  
  

```
{  
    "Version": "2012-10-17",  
    "Statement": \[{  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "arn:aws:iam::111111111111:role/market-s3-role"  
            },  
            "Action": \[  
                "s3:ListBucket",  
                "s3:GetBucketLocation"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::market-s3-bucket"  
            \]},  
        {  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "arn:aws:iam::1111111111:role/market-s3-role"  
            },  
            "Action": \[  
                "s3:GetObject",  
                "s3:PutObject",  
                "s3:DeleteObject"  
            \],  
            "Resource": \[  
                "arn:aws:s3:::market-s3-bucket/\*"  
            \]  
        }  
    \]  
}  

```

  
Once the bucket policy is in place, we can now test the access from the EC2 instance in IT account. Open CloudBerry Drive and mount the S3 bucket as G: drive. You should be able to see, create and delete files/folders in the drive.  
  
Note: In this setup we used IAM Resource Based Policy to gain access cross account AWS resources. This is the easiest way in our scenario. But not all AWS resource support Resource Based Policy. For resources do not support, we will need to use Cross-account IAM roles. You can read more from [here](https://aws.amazon.com/premiumsupport/knowledge-center/cross-account-access-s3/).