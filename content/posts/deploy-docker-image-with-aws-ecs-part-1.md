---
title: 'Deploy Docker Image with AWS ECS (Part 1)'
date: 2018-08-24T13:00:00.000-07:00
draft: false
url: /2018/08/deploy-docker-image-with-aws-ecs-part-1.html
---

One of the things I have been working on is to help our developers to containerize their applications and deploy them to AWS ECS.   
In this post, I will walk through the steps to upload a Docker image to AWS ECR (Elastic Container Repository).  
As the first step, we need to provision the ECR with CloudFormation template.  
Below is a simple CFN template written in YAML.  
  

```yaml
AWSTemplateFormatVersion: "2010-09-09"  
  
Description: \>  
  Play stack  
  
Parameters:  
  RepoName:  
    Default: tomrepo  
    Description: ECR Repoistory Name       
    Type: String     \# required  
    ConstraintDescription: must be a name  
  
Resources:  
  myrepo:  
    Type: AWS::ECR::Repository  
    Properties:  
      RepositoryName: !Ref RepoName       
  mycluster:  
    Type: AWS::ECS::Cluster  
    Properties:  
      ClusterName: tomecscluster       
Outputs:  
  AWSTemplateFormatVersion:  
    Description: Tempalte version  
    Value: "1.0"  

```

Deploy the CFN template with AWS CLI command below.  
  

```bash
aws cloudformation create-stack --stack-name createecs --template-body file://mytemplate.yaml --parameters ParameterKey=RepoName,ParameterValue=webfront  
```

After the template is successfully deployed, go to ECS in AWS Console and confirm the repository and cluster is created.  
  
[![image](https://lh3.googleusercontent.com/-FHfR1n5QJZk/W4Bu12q0sjI/AAAAAAAAKOQ/55V03QsV8cYOJfNmT-lW7YRZ8yFf0KDXgCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-COhxC5Zzdyc/W4Bu0A6hxZI/AAAAAAAAKOM/w3-Fz2iP6Z4WnlmI8mpP9o7jCH34-nu7gCHMYCw/s1600-h/image2)  
[![image](https://lh3.googleusercontent.com/-Bn_yNuO_evo/W4Bu4LiU8vI/AAAAAAAAKOY/d655VtLTPK0_C7yPw72-5OIQwONfz2eYgCHMYCw/image_thumb2?imgmax=800 "image")](https://lh3.googleusercontent.com/-Ka6pWXgfFNA/W4Bu23wlw3I/AAAAAAAAKOU/a61y55PUPXg0mfCeRfGVhSNnikET1v2awCHMYCw/s1600-h/image8)  
Next we need to build the Docker image locally to be used for this deployment. In this example I use my local Docker engine installed on my Win 10 workstation. You can learn more about Docker for Windows [here](https://docs.docker.com/docker-for-windows/).  
Once Docker is installed and running, you should see a little docker icon in the task tray. Make sure it is switched to Linux Containers.  
[![image](https://lh3.googleusercontent.com/-TtMI-MYdcKI/W4Bu6V2UEeI/AAAAAAAAKOg/XZhQZriiKKMXvMLvrfuxTPeEYUPJ8K9iACHMYCw/image_thumb3?imgmax=800 "image")](https://lh3.googleusercontent.com/-EFUJu1rdWlM/W4Bu5AAtZzI/AAAAAAAAKOc/zSp9NIqfwUISXbExqIbj6SHEoAd87FCpwCHMYCw/s1600-h/image11)  
Open CMD and create a folder to host the docker image.  
Create a file named as “Dockfile” in the folder. Yes, it does not have an extension in the file name.  
Copy the code below to the file. The Dockerfile basically uses a Ubuntu image and installed Apache web service on it along with a simple webpage named index.html.  
  

```Dockerfile
FROM ubuntu:12.04  
  
# Install dependencies  
RUN apt-get update -y  
RUN apt-get install -y apache2  
  
# Install apache and write hello world message  
RUN echo "Hello World!" > /var/www/index.html  
  
# Configure apache  
RUN a2enmod rewrite  
RUN chown -R www-data:www-data /var/www  
ENV APACHE\_RUN\_USER www-data  
ENV APACHE\_RUN\_GROUP www-data  
ENV APACHE\_LOG\_DIR /var/log/apache2  
  
EXPOSE 80  
  
CMD \["/usr/sbin/apache2", "-D",  "FOREGROUND"\]  

```

Save the file and we can now build the image.  
  

```bash
docker build -t webserver .
```

Before we upload the image to ECR, we need to test it out locally first.  
Before you run the image (create the container), make sure port 80 on your local machine is not used.  
  

```bash
docker run -p 80:80 webserver  
```

Now, test access to [http://localhost](http://localhost/), you should see the “Hello World” page.  
[![Image](https://lh3.googleusercontent.com/-i3nXESQAcss/W4Bu8D16b2I/AAAAAAAAKOo/4j4-TAtiP808GXvQp0InEI8zQTvXTkNYwCHMYCw/Image_thumb%255B1%255D?imgmax=800 "Image")](https://lh3.googleusercontent.com/-Yng9YJFvA_M/W4Bu7KLRhyI/AAAAAAAAKOk/Wle0ovT7KfUZM8N3laPetigNEQb8FsXAQCHMYCw/s1600-h/Image%255B3%255D)  
The image is now ready to be pushed to AWS ECR.  
First, we need to retrieve the login information to authenticate our local Docker client with ECR. Run the command below and **COPY the output.** Change the region parameter if your ECR is in a different region.  
  

```bash
aws ecr get-login --no-include-email --region ap-southeast-1  
```

The output of the command should look like something below.  
[![image](https://lh3.googleusercontent.com/-YRX2xPH3zTI/W4Bu9-IXmnI/AAAAAAAAKOw/dvVbGCibbgQPZh2UvlMpJNRu-86Ukr2qACHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-4b9RT79-dDc/W4Bu8-oTKuI/AAAAAAAAKOs/2rAcor_I90wVrD6bFnw-5oOJAEfOTNX2ACHMYCw/s1600-h/image%255B6%255D)  
Copy the whole output and paste it into command line. Press enter to execute it.  
You should get output like below, indicating your Docker client is now authenticated with AWS ECR.  
[![Image](https://lh3.googleusercontent.com/-gKb1zqenPPs/W4BvAFtz6JI/AAAAAAAAKO4/2MTZnsYNZyEMEzpmK3Y-2aiCcs7vJ3JyACHMYCw/Image_thumb%255B3%255D?imgmax=800 "Image")](https://lh3.googleusercontent.com/-GIP_Q6cKQoQ/W4Bu-sUZT1I/AAAAAAAAKO0/BTZr-y8mJukVijL9TGWeLjqaNXmPo3p6QCHMYCw/s1600-h/Image%255B9%255D)  
Tag the image you got and make it ready for the push. If you run docker images, you should see a new image with aws tag  
[![Image(8)](https://lh3.googleusercontent.com/-Cd3DPbOb3Ns/W4BvCd9sfjI/AAAAAAAAKPA/tUnVjjymTP8ytNLMSdgaGZTrkFjDC1AwQCHMYCw/Image%25288%2529_thumb%255B2%255D?imgmax=800 "Image(8)")](https://lh3.googleusercontent.com/-54opLlWhbgU/W4BvBRTPv2I/AAAAAAAAKO8/L12b2JDn34s5ZJQeqSVPz8lM5ny0myAwwCHMYCw/s1600-h/Image%25288%2529%255B5%255D)  
You can now push the image.  
  

```bash
docker push 1234567891011.dkr.ecr.ap-southeast-1.amazonaws.com/webfront:latest  
```

It may take sometime for the image to be uploaded. But eventually you should see something like below.  
[![Image](https://lh3.googleusercontent.com/-pGvW8cE4-4o/W4BvEn4QhgI/AAAAAAAAKPM/ZciAL3XjYxIAbiWedCnxPCgJHMc-w64SgCHMYCw/Image_thumb%255B5%255D?imgmax=800 "Image")](https://lh3.googleusercontent.com/-2nsci46X3tA/W4BvDXmDadI/AAAAAAAAKPE/FHwvDKiefrEqjqe4mMv16TCJCmUWgQUWACHMYCw/s1600-h/Image%255B13%255D)  
In AW Console, under the ECR “webfront” repository, you should now see the image tagged as latest.  
[![image](https://lh3.googleusercontent.com/-_SUf23sxBgo/W4BvGQkx67I/AAAAAAAAKPU/pmoWUFoNtYksSOToCR4yEB7K9TO4yrSRACHMYCw/image_thumb%255B6%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-5ro2utj5Se4/W4BvFv9m-eI/AAAAAAAAKPQ/UwnXCfYrh7snOLXyVXU6OBtQnEcg_ECJgCHMYCw/s1600-h/image%255B16%255D)