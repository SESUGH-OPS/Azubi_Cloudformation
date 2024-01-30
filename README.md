# *AZUBI CLOUDFORMATION*
**This task entails on automating resourses using cloud formation on AWS**

## GUIDELINES:-

*Below are the steps to automate resources on cloud formation*

### STEP 1:- DEPLOY A CLOUD FORMATION STACK
- *Copy the below cloudformation stack code and open in a code  editor such as nano,vim or vscode*

```
AWSTemplateFormatVersion: 2010-09-09
Description: Lab template

# Lab VPC with public subnet and Internet Gateway

Parameters:

  LabVpcCidr:
    Type: String
    Default: 10.0.0.0/20

  PublicSubnetCidr:
    Type: String
    Default: 10.0.0.0/24


Resources:

  MyS3Bucket:
    Type: AWS::S3::Bucket

###########
# EC2
###########
  LabInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      KeyName: mykeypair
      NetworkInterfaces:
      - AssociatePublicIpAddress: 'true'
        DeviceIndex: '0'
        GroupSet:
        - sg-09d6d2401eed7ade3
      SubnetId: !Ref PublicSubnet

###########
# VPC with Internet Gateway
###########

  LabVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref LabVpcCidr
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Lab VPC

  IGW:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Lab IGW

  VPCtoIGWConnection:
    Type: AWS::EC2::VPCGatewayAttachment
    DependsOn:
      - IGW
      - LabVPC
    Properties:
      InternetGatewayId: !Ref IGW
      VpcId: !Ref LabVPC

###########
# Public Route Table
###########

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    DependsOn: LabVPC
    Properties:
      VpcId: !Ref LabVPC
      Tags:
        - Key: Name
          Value: Public Route Table

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn:
      - PublicRouteTable
      - IGW
    Properties:
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref IGW
      RouteTableId: !Ref PublicRouteTable

###########
# Public Subnet
###########

  PublicSubnet:
    Type: AWS::EC2::Subnet
    DependsOn: LabVPC
    Properties:
      VpcId: !Ref LabVPC
      MapPublicIpOnLaunch: true
      CidrBlock: !Ref PublicSubnetCidr
      AvailabilityZone: !Select 
        - 0
        - !GetAZs 
          Ref: AWS::Region
      Tags:
        - Key: Name
          Value: Public Subnet

  PublicRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    DependsOn:
      - PublicRouteTable
      - PublicSubnet
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet

###########
# App Security Group
###########

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    DependsOn: LabVPC
    Properties:
      GroupName: App
      GroupDescription: Enable access to App
      VpcId: !Ref LabVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: App

###########
# Outputs
###########

Outputs:

  LabVPCDefaultSecurityGroup:
    Value: !Sub ${LabVPC.DefaultSecurityGroup}  
 
  ```
### STEP 2:- *GO TO YOUR AWS CONSOLE AND OPEN CLOUD FORMATION SERVICE*
- *Create a new stack on the cloud formation service and name it with any name of your choice*
- *Upload your saved code in .yaml format and deploy. The service should create a vpc,an s3 bucket and EC2 instance from your code format uploaded
  You should see a uploaded status like the image below*
  ![console](https://github.com/SESUGH-OPS/Google_Recreate/assets/105423735/d754d306-e81e-4681-a916-fecd20d0ceed)


### STEP 3:- *DESTROY YOUR SERVICE AFTER YOUR PRACTICE UPLOAD TO AVOID COSTS*
  

