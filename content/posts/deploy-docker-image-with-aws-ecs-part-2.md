---
title: 'Deploy Docker Image with AWS ECS (Part 2)'
date: 2018-10-16T20:34:00.000-07:00
draft: false
url: /2018/10/deploy-docker-image-with-aws-ecs-part-2.html
---

In Part 1 we uploaded a Docker image to AWS ECR. In this post, we will complete building the ECS Cluster and deploy the container image onto the cluster.

Note: The lab I worked on was recreated. The container image was renamed from webfront to testweb.

Before we start, you need to understand some ECS basic concepts.

**Task Definition**

A task definition describes one or more containers, their relationships, how they should be launched etc. It’s basically a JSON file contains the configuration details of the container(s).

**Task**

A task is the instantiation of a task definition. They are created based on the task definitions you provided.

**Service (Scheduler)**

In short, the service or service scheduler controls the tasks running across the ECS cluster.

**Cluster**

Cluster is the mothership of those tasks. It hosts the containers launched from the tasks.

**Container Instance**

By now, AWS ECS offers two types of ECS Cluster: Fargate and EC2. With Fargate, AWS manages the cluster resources for you. With EC2, ECS provisions EC2 instances based on your specification and you are in charge of maintaining those EC2 instances. Those EC2 instances are “Container Instances” as they are the ones host containers.

**Container Agent**

For a EC2 ECS cluster, container agent runs on each Container Instance. It is the key component that controls ECS tasks and resource utilization.

To learn more about ECS basic concepts, you can refer to [this AWS document](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html).

**1. Create Task Definition**

The first step we take is to create a new Task Definition.

Choose **EC2** as the launch type.

Name the Task Definition as **testweb-task**.

Leave Network Mode as <default>, which will be Bridged for Linux instance.

Set Task size as below.

[![image](https://lh3.googleusercontent.com/-ri9BQXFJzCc/W8atr_Gx4II/AAAAAAAAKVg/Dl-794wWZ18mFamYfMktaSDwLu7NhmuEQCHMYCw/image_thumb?imgmax=800 "image")](https://lh3.googleusercontent.com/-rQH3Hs9Icrg/W8atqJFEMNI/AAAAAAAAKVc/MG8tdbCLL-swdTOcbDtxS9HCES4RZN4sACHMYCw/s1600-h/image%255B2%255D)

Click **Add container** to add the container image from ECR.

Name the container as **testweb.**

Under **Image**, put in the ECR repo URL with tag, below is an example.

_1234567891011.dkr.ecr.ap-southeast-1.amazonaws.com/testweb:latest_

For memory limits, set hard limit as “512” and soft limit as “256”.

Set Port mappings as 80 to 80 tcp.

Click **Add** to add the container configuration.

Click Create **to create the Task Definition.**

The Task Definition can also be created with JSON. Below is the code.

```json
{  
  "executionRoleArn": null,  
  "containerDefinitions": [  
    {  
      "dnsSearchDomains": null,  
      "logConfiguration": null,  
      "entryPoint": null,  
      "portMappings": [  
        {  
          "hostPort": 80,  
          "protocol": "tcp",  
          "containerPort": 80  
        }  
      ],  
      "command": null,  
      "linuxParameters": null,  
      "cpu": 512,  
      "environment": [],  
      "ulimits": null,  
      "dnsServers": null,  
      "mountPoints": [],  
      "workingDirectory": null,  
      "dockerSecurityOptions": null,  
      "memory": 512,  
      "memoryReservation": 256,  
      "volumesFrom": [],  
      "image": "12345678910.dkr.ecr.ap-southeast-1.amazonaws.com/testweb:latest",  
      "disableNetworking": null,  
      "interactive": null,  
      "healthCheck": null,  
      "essential": true,  
      "links": null,  
      "hostname": null,  
      "extraHosts": null,  
      "pseudoTerminal": null,  
      "user": null,  
      "readonlyRootFilesystem": null,  
      "dockerLabels": null,  
      "systemControls": null,  
      "privileged": null,  
      "name": "testweb"  
    }  
  ],  
  "placementConstraints": [],  
  "memory": "512",  
  "taskRoleArn": null,  
  "compatibilities": [  
    "EC2"  
  ],  
  "taskDefinitionArn": "arn:aws:ecs:ap-southeast-1:297012811963:task-definition/testweb-task:1",  
  "family": "testweb-task",  
  "requiresAttributes": [  
    {  
      "targetId": null,  
      "targetType": null,  
      "value": null,  
      "name": "com.amazonaws.ecs.capability.ecr-auth"  
    },  
    {  
      "targetId": null,  
      "targetType": null,  
      "value": null,  
      "name": "com.amazonaws.ecs.capability.docker-remote-api.1.21"  
    }  
  ],  
  "requiresCompatibilities": [  
    "EC2"  
  ],  
  "networkMode": null,  
  "cpu": "1024",  
  "revision": 1,  
  "status": "ACTIVE",  
  "volumes": []  
}  

```


**2. Create ECS Cluster**

Under Amazon ECS click **Clusters** and click **Create Cluster**.

Select **EC2 Linux + Networking** and click **Next step**.

Name the cluster. I named it as testweb-clu, which is a bad name, you never want to name your cluster with a single image name, as the container suppose to present a micro service of your application.

Choose the EC2 instance type. For testing we will use t2.micro and 1 for Number of instances. Select a keypair so we can log into the Container Instances for some troubleshooting if needed.

[![image](https://lh3.googleusercontent.com/-vvvO-6bwwKo/W8a_Mr1JTpI/AAAAAAAAKWA/evWbE17Hh7sWyAl8BdnItwPN-868KHUVQCHMYCw/image_thumb%255B1%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-6bzT30sFlAg/W8a_LuWhMjI/AAAAAAAAKV8/mz67QJGxFL86XZrSwnIOstbnZdm6rIWbACHMYCw/s1600-h/image%255B5%255D)

Under Networking, Create a new VPC, along with two new subnets.

Create a new security group and add rule to allow public access to TCP port 80.

[![image](https://lh3.googleusercontent.com/-1sxWGaowNGQ/W8a_PKLKEdI/AAAAAAAAKWI/9M6zR9mTmM8uMQZfyZtkiC8-bUOjpr6zACHMYCw/image_thumb%255B2%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-yeAnpBHmek4/W8a_N9_UN_I/AAAAAAAAKWE/rRiiuX4IhRUs-NBbs08oCykAwPBV_HfZQCHMYCw/s1600-h/image%255B8%255D)

To allow the EC2 Container Instance to access ECS, we ill need to assign an IAM role to it. The IAM role contains 2 Amazon policies.

[![image](https://lh3.googleusercontent.com/-IthuLqwSjH8/W8a_RVyc3FI/AAAAAAAAKWQ/jb_2WyHF4vglqlDmtTb6w-FWpeQit7OWwCHMYCw/image_thumb%255B3%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-GBamYe8OY5I/W8a_QeMR3pI/AAAAAAAAKWM/TUPjevgfa041G2wVCDzhjCnw1CGZ0Fu2gCHMYCw/s1600-h/image%255B11%255D)

Click **Create** to create the Cluster. Interestingly, you can see the cluster provision is actually done through a CloudFormation template.

[![image](https://lh3.googleusercontent.com/-iJ3IW2oxYJE/W8a_TkldYbI/AAAAAAAAKWY/WkJPPAwDhnwmICrj_km9-VYCiw-Zg-M9gCHMYCw/image_thumb%255B4%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-vtmnPz7P8oM/W8a_StkvyAI/AAAAAAAAKWU/BjxeEnO4tpgrMrWTKUzWTWt15NCd44ZRACHMYCw/s1600-h/image%255B14%255D)

From the CloudFormation stack details, we can see all the script did was to provision a Launch Configuration and a AutoScaling Group.

[![image](https://lh3.googleusercontent.com/-psk9l27yWCU/W8a_VgK2cUI/AAAAAAAAKWg/lFKfkUimUSEPLyz6KmTyLxYwx_bOlNbdgCHMYCw/image_thumb%255B5%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-9iyQWPRL-O4/W8a_UlsAPfI/AAAAAAAAKWc/x65lG0CBex0DP5n09GytWr80gINP-07hwCHMYCw/s1600-h/image%255B17%255D)

Here is the CloudFormation Template code.

```yaml
AWSTemplateFormatVersion: '2010-09-09'  
Description: \>  
  AWS CloudFormation template to create a new VPC  
  or use an existing VPC for ECS deployment  
  in Create Cluster Wizard. Requires exactly 1  
  Instance Types for a Spot Request.  
Parameters:  
  EcsClusterName:  
    Type: String  
    Description: \>  
      Specifies the ECS Cluster Name with which the resources would be  
      associated  
    Default: default  
  EcsAmiId:  
    Type: String  
    Description: Specifies the AMI ID for your container instances.  
  EcsInstanceType:  
    Type: CommaDelimitedList  
    Description: \>  
      Specifies the EC2 instance type for your container instances.  
      Defaults to m4.large  
    Default: m4.large  
    ConstraintDescription: must be a valid EC2 instance type.  
  KeyName:  
    Type: String  
    Description: \>  
      Optional - Specifies the name of an existing Amazon EC2 key pair  
      to enable SSH access to the EC2 instances in your cluster.  
    Default: ''  
  VpcId:  
    Type: String  
    Description: \>  
      Optional - Specifies the ID of an existing VPC in which to launch  
      your container instances. If you specify a VPC ID, you must specify a list of  
      existing subnets in that VPC. If you do not specify a VPC ID, a new VPC is created  
      with atleast 1 subnet.  
    Default: ''  
    ConstraintDescription: \>  
      VPC Id must begin with 'vpc-' or leave blank to have a  
      new VPC created  
  SubnetIds:  
    Type: CommaDelimitedList  
    Description: \>  
      Optional - Specifies the Comma separated list of existing VPC Subnet  
      Ids where ECS instances will run  
    Default: ''  
  SecurityGroupId:  
    Type: String  
    Description: \>  
      Optional - Specifies the Security Group Id of an existing Security  
      Group. Leave blank to have a new Security Group created  
    Default: ''  
  VpcCidr:  
    Type: String  
    Description: Optional - Specifies the CIDR Block of VPC  
    Default: ''  
  SubnetCidr1:  
    Type: String  
    Description: Specifies the CIDR Block of Subnet 1  
    Default: ''  
  SubnetCidr2:  
    Type: String  
    Description: Specifies the CIDR Block of Subnet 2  
    Default: ''  
  SubnetCidr3:  
    Type: String  
    Description: Specifies the CIDR Block of Subnet 3  
    Default: ''  
  AsgMaxSize:  
    Type: Number  
    Description: \>  
      Specifies the number of instances to launch and register to the cluster.  
      Defaults to 1.  
    Default: '1'  
  IamRoleInstanceProfile:  
    Type: String  
    Description: \>  
      Specifies the Name or the Amazon Resource Name (ARN) of the instance  
      profile associated with the IAM role for the instance  
  SecurityIngressFromPort:  
    Type: Number  
    Description: \>  
      Optional - Specifies the Start of Security Group port to open on  
      ECS instances - defaults to port 0  
    Default: '0'  
  SecurityIngressToPort:  
    Type: Number  
    Description: \>  
      Optional - Specifies the End of Security Group port to open on ECS  
      instances - defaults to port 65535  
    Default: '65535'  
  SecurityIngressCidrIp:  
    Type: String  
    Description: \>  
      Optional - Specifies the CIDR/IP range for Security Ports - defaults  
      to 0.0.0.0/0  
    Default: 0.0.0.0/0  
  EcsEndpoint:  
    Type: String  
    Description: \>  
      Optional - Specifies the ECS Endpoint for the ECS Agent to connect to  
    Default: ''  
  VpcAvailabilityZones:  
    Type: CommaDelimitedList  
    Description: \>  
      Specifies a comma-separated list of 3 VPC Availability Zones for  
      the creation of new subnets. These zones must have the available status.  
    Default: ''  
  EbsVolumeSize:  
    Type: Number  
    Description: \>  
      Optional - Specifies the Size in GBs, of the newly created Amazon  
      Elastic Block Store (Amazon EBS) volume  
    Default: '0'  
  EbsVolumeType:  
    Type: String  
    Description: Optional - Specifies the Type of (Amazon EBS) volume  
    Default: ''  
    AllowedValues:  
      - ''  
      - standard  
      - io1  
      - gp2  
      - sc1  
      - st1  
    ConstraintDescription: Must be a valid EC2 volume type.  
  DeviceName:  
    Type: String  
    Description: Optional - Specifies the device mapping for the Volume  
  UseSpot:  
    Type: String  
    Default: 'false'  
  IamSpotFleetRoleArn:  
    Type: String  
    Default: ''  
  SpotPrice:  
    Type: String  
    Default: ''  
  SpotAllocationStrategy:  
    Type: String  
    Default: 'diversified'  
    AllowedValues:  
      - 'lowestPrice'  
      - 'diversified'  
  UserData:  
    Type: String  
  IsWindows:  
    Type: String  
    Default: 'false'  
Conditions:  
  CreateEC2LCWithKeyPair:  
    !Not [!Equals [!Ref KeyName, '']]  
  SetEndpointToECSAgent:  
    !Not [!Equals [!Ref EcsEndpoint, '']]  
  CreateNewSecurityGroup:  
    !Equals [!Ref SecurityGroupId, '']  
  CreateNewVpc:  
    !Equals [!Ref VpcId, '']  
  CreateSubnet1: !And  
    - !Not [!Equals [!Ref SubnetCidr1, '']]  
    - !Condition CreateNewVpc  
  CreateSubnet2: !And  
    - !Not [!Equals [!Ref SubnetCidr2, '']]  
    - !Condition CreateSubnet1  
  CreateSubnet3: !And  
    - !Not [!Equals [!Ref SubnetCidr3, '']]  
    - !Condition CreateSubnet2  
  CreateWithSpot: !Equals [!Ref UseSpot, 'true']  
  CreateWithASG: !Not [!Condition CreateWithSpot]  
  CreateWithSpotPrice: !Not [!Equals [!Ref SpotPrice, '']]  
Resources:  
  Vpc:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::VPC  
    Properties:  
      CidrBlock: !Ref VpcCidr  
      EnableDnsSupport: 'true'  
      EnableDnsHostnames: 'true'  
  PubSubnetAz1:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::Subnet  
    Properties:  
      VpcId: !Ref Vpc  
      CidrBlock: !Ref SubnetCidr1  
      AvailabilityZone: !Select [ 0, !Ref VpcAvailabilityZones ]  
      MapPublicIpOnLaunch: true  
  PubSubnetAz2:  
    Condition: CreateSubnet2  
    Type: AWS::EC2::Subnet  
    Properties:  
      VpcId: !Ref Vpc  
      CidrBlock: !Ref SubnetCidr2  
      AvailabilityZone: !Select [ 1, !Ref VpcAvailabilityZones ]  
      MapPublicIpOnLaunch: true  
  PubSubnetAz3:  
    Condition: CreateSubnet3  
    Type: AWS::EC2::Subnet  
    Properties:  
      VpcId: !Ref Vpc  
      CidrBlock: !Ref SubnetCidr3  
      AvailabilityZone: !Select [ 2, !Ref VpcAvailabilityZones ]  
      MapPublicIpOnLaunch: true  
  InternetGateway:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::InternetGateway  
  AttachGateway:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::VPCGatewayAttachment  
    Properties:  
      VpcId: !Ref Vpc  
      InternetGatewayId: !Ref InternetGateway  
  RouteViaIgw:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::RouteTable  
    Properties:  
      VpcId: !Ref Vpc  
  PublicRouteViaIgw:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::Route  
    DependsOn: AttachGateway  
    Properties:  
      RouteTableId: !Ref RouteViaIgw  
      DestinationCidrBlock: 0.0.0.0/0  
      GatewayId: !Ref InternetGateway  
  PubSubnet1RouteTableAssociation:  
    Condition: CreateSubnet1  
    Type: AWS::EC2::SubnetRouteTableAssociation  
    Properties:  
      SubnetId: !Ref PubSubnetAz1  
      RouteTableId: !Ref RouteViaIgw  
  PubSubnet2RouteTableAssociation:  
    Condition: CreateSubnet2  
    Type: AWS::EC2::SubnetRouteTableAssociation  
    Properties:  
      SubnetId: !Ref PubSubnetAz2  
      RouteTableId: !Ref RouteViaIgw  
  PubSubnet3RouteTableAssociation:  
    Condition: CreateSubnet3  
    Type: AWS::EC2::SubnetRouteTableAssociation  
    Properties:  
      SubnetId: !Ref PubSubnetAz3  
      RouteTableId: !Ref RouteViaIgw  
  EcsSecurityGroup:  
    Condition: CreateNewSecurityGroup  
    Type: AWS::EC2::SecurityGroup  
    Properties:  
      GroupDescription: ECS Allowed Ports  
      VpcId: !If [ CreateSubnet1, !Ref Vpc, !Ref VpcId ]  
      SecurityGroupIngress:  
        IpProtocol: tcp  
        FromPort: !Ref SecurityIngressFromPort  
        ToPort: !Ref SecurityIngressToPort  
        CidrIp: !Ref SecurityIngressCidrIp  
  EcsInstanceLc:  
    Type: AWS::AutoScaling::LaunchConfiguration  
    Condition: CreateWithASG  
    Properties:  
      ImageId: !Ref EcsAmiId  
      InstanceType: !Select [ 0, !Ref EcsInstanceType ]  
      AssociatePublicIpAddress: true  
      IamInstanceProfile: !Ref IamRoleInstanceProfile  
      KeyName: !If [ CreateEC2LCWithKeyPair, !Ref KeyName, !Ref "AWS::NoValue" ]  
      SecurityGroups: [ !If [ CreateNewSecurityGroup, !Ref EcsSecurityGroup, !Ref SecurityGroupId ] ]  
      BlockDeviceMappings:  
      - DeviceName: !Ref DeviceName  
        Ebs:  
         VolumeSize: !Ref EbsVolumeSize  
         VolumeType: !Ref EbsVolumeType  
      UserData:  
        Fn::Base64: !Ref UserData  
  EcsInstanceAsg:  
    Type: AWS::AutoScaling::AutoScalingGroup  
    Condition: CreateWithASG  
    Properties:  
      VPCZoneIdentifier: !If  
        - CreateSubnet1  
        - !If  
          - CreateSubnet2  
          - !If  
            - CreateSubnet3  
            - [ !Sub "${PubSubnetAz1},  ${PubSubnetAz2},  ${PubSubnetAz3}" ]  
            - [ !Sub "${PubSubnetAz1},  ${PubSubnetAz2}" ]  
          - [ !Sub "${PubSubnetAz1}" ]  
        - !Ref SubnetIds  
      LaunchConfigurationName: !Ref EcsInstanceLc  
      MinSize: '0'  
      MaxSize: !Ref AsgMaxSize  
      DesiredCapacity: !Ref AsgMaxSize  
      Tags:  
        -  
          Key: Name  
          Value: !Sub "ECS  Instance  -  ${AWS::StackName}"  
          PropagateAtLaunch: 'true'  
        -  
          Key: Description  
          Value: "This  instance  is  the  part  of  the  Auto  Scaling  group  which  was  created  through  ECS  Console"  
          PropagateAtLaunch: 'true'  
  EcsSpotFleet:  
    Condition: CreateWithSpot  
    Type: AWS::EC2::SpotFleet  
    Properties:  
      SpotFleetRequestConfigData:  
        AllocationStrategy: !Ref SpotAllocationStrategy  
        IamFleetRole: !Ref IamSpotFleetRoleArn  
        TargetCapacity: !Ref AsgMaxSize  
        SpotPrice: !If [ CreateWithSpotPrice, !Ref SpotPrice, !Ref 'AWS::NoValue' ]  
        TerminateInstancesWithExpiration: true  
        LaunchSpecifications:   
            -  
              IamInstanceProfile:  
                Arn: !Ref IamRoleInstanceProfile  
              ImageId: !Ref EcsAmiId  
              InstanceType: !Select [ 0, !Ref EcsInstanceType ]  
              KeyName: !If [ CreateEC2LCWithKeyPair, !Ref KeyName, !Ref "AWS::NoValue" ]  
              Monitoring:  
                Enabled: true  
              SecurityGroups:  
                - GroupId: !If [ CreateNewSecurityGroup, !Ref EcsSecurityGroup, !Ref SecurityGroupId ]  
              SubnetId: !If  
                      - CreateSubnet1  
                      - !If  
                        - CreateSubnet2  
                        - !If  
                          - CreateSubnet3  
                          - !Join [ "," , [ !Ref PubSubnetAz1, !Ref PubSubnetAz2, !Ref PubSubnetAz3 ] ]  
                          - !Join [ "," , [ !Ref PubSubnetAz1, !Ref PubSubnetAz2 ] ]  
                        - !Ref PubSubnetAz1  
                      - !Join [ "," , !Ref SubnetIds ]  
              BlockDeviceMappings:  
                    - DeviceName: !Ref DeviceName  
                      Ebs:  
                       VolumeSize: !Ref EbsVolumeSize  
                       VolumeType: !Ref EbsVolumeType  
              UserData:  
                    Fn::Base64: !Ref UserData  
Outputs:  
  EcsInstanceAsgName:  
    Condition: CreateWithASG  
    Description: Auto Scaling Group Name for ECS Instances  
    Value: !Ref EcsInstanceAsg  
  EcsSpotFleetRequestId:  
      Condition: CreateWithSpot  
      Description: Spot Fleet Request for ECS Instances  
      Value: !Ref EcsSpotFleet  
  UsedByECSCreateCluster:  
    Description: Flag used by ECS Create Cluster Wizard  
    Value: 'true'  
  TemplateVersion:  
    Description: The version of the template used by Create Cluster Wizard  
    Value: '2.0.0'  

```

Here is the parameters for the template

AsgMaxSize: 1

DeviceName: /dev/xvdcz

EbsVolumeSize: 22

EbsVolumeType: gp2

EcsAmiId: ami-0a3f70f0111af1d29

EcsClusterName: testweb-clu

EcsEndpoint:

EcsInstanceType: t2.micro

IamRoleInstanceProfile: arn:aws:iam::123456789103:instance-profile/ecsInstanceRole

IamSpotFleetRoleArn:

IsWindows: false

KeyName: tom-testkey

SecurityGroupId: sg-014e7a96be118e111

SecurityIngressCidrIp: 0.0.0.0/0

SecurityIngressFromPort: 80

SecurityIngressToPort: 80

SpotAllocationStrategy: diversified

SpotPrice:

SubnetCidr1: 10.0.0.0/24

SubnetCidr2: 10.0.1.0/24

SubnetCidr3:

SubnetIds: subnet-057b8e2cd11183e28

UserData: #!/bin/bash echo ECS_CLUSTER=testweb-clu >> /etc/ecs/ecs.config;echo ECS_BACKEND_HOST= >> /etc/ecs/ecs.config;

UseSpot: false

VpcAvailabilityZones: ap-southeast-1a,ap-southeast-1b,ap-southeast-1c

VpcCidr: 10.0.0.0/16

VpcId: vpc-08d111d48c2f68111

**3. Create Service**

Back in ECS, click the Cluster name.

Under Services tab, click Create.

Choose EC2 as the Launch Type, select the Task Definition we created in step1, select the cluster we created in step 2. Set **Number of tasks** to 1.

Use **AZ Balanced Spread** for **Task Placement**.

[![image](https://lh3.googleusercontent.com/-VqrZbvSBVpo/W8a_YhkXZ2I/AAAAAAAAKWo/aIITCq8C81A9wIX04xLJzVL7WMUO81ldQCHMYCw/image_thumb%255B6%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-IS4BEcRirTo/W8a_XF8zl1I/AAAAAAAAKWk/Hp6ATq-rcoIW4pngMwrZIr4XacCdOqxawCHMYCw/s1600-h/image%255B20%255D)

Leave everything unchanged in **Configure network**.

Leave AutoScaling as “Do not adjust…”.

Click **Create Service** to create the service.

**4. Test**

Once the service is successfully created, The cluster will then start to provision the Container Instance and deploy the container image onto it. This process does seem to take time. In my case, it took around 55 minutes for the container to be started! So you definitely want to warm up your cluster in production.

[![image](https://lh3.googleusercontent.com/-hyMAslr4nqE/W8a_avbWPpI/AAAAAAAAKWw/ZLJ289pC2lksaQ_LJmJGpNWiI29dCftxQCHMYCw/image_thumb%255B7%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-hCZOFQmxD38/W8a_Zo1scJI/AAAAAAAAKWs/5FjRLEmT8T87T-9ElHHIzQLw8A7zQ1bQQCHMYCw/s1600-h/image%255B23%255D)

Under Tasks, you should see a running task there.

[![image](https://lh3.googleusercontent.com/-zdvoDS4eG1M/W8a_cuux0iI/AAAAAAAAKW4/DeOPv5WeIewCn0guhW7Whxev19MXOtW9ACHMYCw/image_thumb%255B8%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-wTWwbrHucLA/W8a_bZrGM9I/AAAAAAAAKW0/2kQUOsimCt0Gd47JmV8BVL7Rl_IA260LgCHMYCw/s1600-h/image%255B26%255D)

Go to EC2 and find the public IP of the Container Instance.

[![image](https://lh3.googleusercontent.com/-M2MJ7-EfpKc/W8a_fDyzrBI/AAAAAAAAKXA/zZQ-V24F9dcGIjlKhtB7_rVLKdsILL5ZQCHMYCw/image_thumb%255B9%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-OH7PtlYaeoI/W8a_dgmJzCI/AAAAAAAAKW8/vwJ86ngNoY0eA8sgvGIjoYtEMpS1HT1QQCHMYCw/s1600-h/image%255B29%255D)

In browser, type [http://publicIP](http://publicIP) and you should see the boring but exciting Hello World message!

[![image](https://lh3.googleusercontent.com/-7pF37V9EmUk/W8a_hY9w4zI/AAAAAAAAKXM/XOk9GOgnEGYWPrP53PkMEXgl3ecK3m29gCHMYCw/image_thumb%255B10%255D?imgmax=800 "image")](https://lh3.googleusercontent.com/-HCtt7aR0V0o/W8a_gZTIUTI/AAAAAAAAKXE/PlfRNdumZk0iaV3EjvHZBTV8Ij6sqiqDACHMYCw/s1600-h/image%255B32%255D)

In case you run into any issues, like the task refuse to launch, one place to look is the ECS Agent log on the Container Instance. The log can be found at /var/logs.