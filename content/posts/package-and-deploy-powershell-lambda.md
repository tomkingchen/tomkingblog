---
title: 'Package and deploy a PowerShell Lambda function with custom modules'
date: 2020-06-06T05:20:00.003-07:00
draft: false
url: /2020/06/package-and-deploy-powershell-lambda.html
tags: 
- PowerShell
- AWS
---

Recently I had the need to create a Lambda function with PowerShell 7. The function is to synchronize data between two REST APIs. It's fairly simple, but does need to use a custom made module. I spent quite bit time to find out how to deploy PowerShell Lambdas with custom modules. Thought might write a guide to help people want to do the same.   

  

My script is fairly simple, it gets a list of users from one API and then convert it to a XML format object and export into the target API. To reuse some of the code, The script requires AWS.Tools module as well as a custom made module, let's call it CAT. The CAT is a module contains some functions that are invoked by the main script. The inital lines of my main script look like below:

```
#Requires -Modules @{ModuleName='AWS.Tools.Common';ModuleVersion='4.0.5.0'}  
#Requires -Modules @{ModuleName='AWS.Tools.SecretsManager';ModuleVersion='4.0.5.0'}  
#Requires -Modules CAT 
```

In addition to install the **AWS.Tools** modules, I manually copied my CAT module folder to the **$Env:PSModulePath**.

  

The nex thing is to package the code into a zip file along with all required modules. To do so, install **AWSLambdaPSCore** module and run this PowerShell command:

```
New-AWSPowerShellLambdaPackage -ScriptPath "../functions/FunctionName.ps1" -outputPackage "FunctionName.zip"
```

You should see the script and modules are all packaged as shown below.

```
Staging deployment at C:\\Users\\tom\\AppData\\Local\\Temp\\myfunction  
Configuring PowerShell to version 7.0.0  
Generating C# project C:\\Users\\tom\\AppData\\Local\\Temp\\myfunction\\myfunction.csproj used to create Lambda function bundle.  
Generating C:\\Users\\tom\\AppData\\Local\\Temp\\myfunction\\Bootstrap.cs to load PowerShell script and required modules in Lambda environment.  
Generating aws-lambda-tools-defaults.json config file with default values used when publishing project.  
Copying PowerShell script to staging directory  
Copying local module AWS.Tools.Common(4.0.5.0) from C:\\Users\\tom\\OneDrive\\Documents\\PowerShell\\Modules\\AWS.Tools.Common\\4.0.5.0  
Copying local module AWS.Tools.SecretsManager(4.0.5.0) from C:\\Users\\tom\\OneDrive\\Documents\\PowerShell\\Modules\\AWS.Tools.SecretsManager\\4.0.5.0  
Copying local module CAT(0.0.2) from C:\\program files\\powershell\\7\\Modules\\CAT  
Resolved full output package path as C:\\Users\\tom\\Dropbox\\github\\myproject\\deploy\\myfunction.zip  
Creating deployment package at C:\\Users\\tom\\Dropbox\\github\\myproject\\deploy\\myfunction.zip  
Restoring .NET Lambda deployment tool  
Initiate packaging  
When deploying this package to AWS Lambda you will need to specify the function handler. The handler for this package is: myfunction::myfunction.Bootstrap::ExecuteFunction. To request Lambda to invoke a specific PowerShell function in your script specify the name of the PowerShell function in the environment variable AWS\_POWERSHELL\_FUNCTION\_HANDLER when publishing the package.  
LambdaHandler                                         PathToPackage                                                         PowerShellFunctionHandlerEnvVar  
\-------------                                         -------------                                                         -------------------------------  
myfunction::myfunction.Bootstrap::ExecuteFunction C:\\Users\\tom\\Dropbox\\github\\myproject\\deploy\\myfunction.zip AWS\_POWERSHELL\_FUNCTION\_HANDLER
```

The steps below are not required if you have a CICD pipeline built up like me. As soon as I push my updated code and package to GitHub, the pipeline will automatically upload the package and deploy it. But if don't, read on! 

  

The next step is to create a S3 bucket to host your package. You can skip this step if you already have a designated bucket for such purpose.

  

After created the bucket, we will then create a **SAM template**. SAM template is basically a transformed CloudFormation template. It allows us to add all the neccessary bits like Events and DeadLetterQueue under AWS::Serverless::Function resource. Here's a simple example. Pay close attention to **CodeUri**. Notice it's pointing to the local zip file instead of a S3 URL? This is by purpose. I will explain in next step.

```
AWSTemplateFormatVersion: '2010-09-09'  
Transform: 'AWS::Serverless-2016-10-31'  
Resources:  
  PopCycler:  
    Type: 'AWS::Serverless::Function'  
    Properties:  
      Handler: myfunction::myfunction.Bootstrap::ExecuteFunction  
      Runtime: dotnetcore3.1  
      **CodeUri: './myfunction.zip'**  
      Role:  
        Fn::GetAtt:  
        - LambdaRole  
        - Arn  
      Environment:  
        Variables:  
          ENV: dev  
          CATURL: https://dev.CAT.com  
          CATAPPID: Abc12345  
          CATSECRETID: CAT-api-key  
          VENDORSECRETID: myproject  
      Events:  
        ScanInterval:  
          Type: Schedule  
          Properties:  
            Schedule: 'rate(1 day)'  
      Timeout: 300  
      MemorySize: 256  
      Tags:  
        owner: tom  
  LambdaRole:  
    Type: 'AWS::IAM::Role'  
    Properties:  
      RoleName: myproject-lambdarole  
      Tags:  
        -   
          Key: 'owner'  
          Value: 'myteam'  
      ManagedPolicyArns:  
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'  
      AssumeRolePolicyDocument:  
        Version: 2012-10-17  
        Statement:  
          - Effect: Allow  
            Principal:  
              Service:  
                - lambda.amazonaws.com  
            Action:  
              - 'sts:AssumeRole'  
      Policies:  
        - PolicyName: MyProjectPolicy  
          PolicyDocument:  
            Version: 2012-10-17  
            Statement:  
              - Effect: Allow  
                Action: secretsmanager:GetSecretValue  
                Resource:  
                  - Fn::Sub: 'arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:CAT-api-key'  
              - Effect: Allow  
                Action: kms:Decrypt  
                Resource:  
                  - Fn::Sub: 'arn:aws:secretsmanager:${AWS::Region}:${AWS::AccountId}:secret:CAT-api-key'
```

After created the template, run the AWS CLI command below. What it does is simply upload the zip file to the specified S3 bucket (myBucket in my example) and update **CodeUri** in your SAM template with the actual S3 URI path to the zip file. Then output a new template for you to use in the next step. You can open up the SAM template **updated.yml** to confirm. 

```
aws cloudformation package --template-file myfunction.yml --s3-bucket mybucket --output-template-file updated.yml
```

The last step is to deploy the Lambda function from **updated.yml**

```
aws cloudformation deploy --stack-name myfunctionstack --template-file updated.yml --capabilities CAPABILITY\_AUTO\_EXPAND CAPABILITY\_NAMED\_IAM
```

That's it folks! I know this isn't as straight forward in comparison to Python and Node.Js runtimes. But as I mentioned before, with a CICD system in place, you should be able to automate the whole deploy process fairly easily. Hope you find it useful!