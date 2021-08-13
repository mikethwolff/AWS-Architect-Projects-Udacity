# AWS Cloud Architect Projects Udacity

The listed projects were part of my AWS Cloud Architect Nanodegree and are displayed in derivated form due to copyright restrictions.

Form and requirements of my projects may differ from present projects. Udacity may vary projects and this repo is representing my findings at this point in time and may not be representative for future  AWS Cloud Architect projects. If you are currently enrolled into Udacity's AWS Cloud Architect Nanodegree please keep in mind that my findings can help you as a guide but copying these findings will go against [Udacity Honor Code](https://udacity.zendesk.com/hc/en-us/articles/210667103-Udacity-Honor-Code).


## Project 1: Design for Availability, Reliability, and Resiliency

In this project you will create highly available solutions to common use cases. You will build a Multi-AvailabilityZone, Multi-Region database and show how to use it in multiple geographically separate AWS regions. You will also build a website hosting solution that is versioned so that any data destruction and accidents can be quickly and easily undone.

There will be three parts to this project:

**Part 1:**
1. Data Durability And Recovery:
- Pick two AWS regions. An active region and a standby region. 
- Use CloudFormation to create one VPC in each region.

**1. Solution:**
vpc.yml
```
Description:  Udacity - Design for Availability, Resilience, and Reliability

Parameters:
  VpcName:
    Description: A name that is prefixed to resource names
    Type: String

  VpcCIDR:
    Description: Please enter the IP range (CIDR notation) for this VPC
    Type: String
    Default: 10.1.0.0/16

  PublicSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the first Availability Zone
    Type: String
    Default: 10.1.10.0/24

  PublicSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the public subnet in the second Availability Zone
    Type: String
    Default: 10.1.11.0/24

  PrivateSubnet1CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the first Availability Zone
    Type: String
    Default: 10.1.20.0/24

  PrivateSubnet2CIDR:
    Description: Please enter the IP range (CIDR notation) for the private subnet in the second Availability Zone
    Type: String
    Default: 10.1.21.0/24

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      -
        Label:
          default: "Information for the VPC"
        Parameters:
          - VpcName
          - VpcCIDR
      -
        Label:
          default: "Information for the VPCs subnets. Subnet network ranges MUST fall within the VPC CIDR block above"
        Parameters:
          - PublicSubnet1CIDR
          - PublicSubnet2CIDR
          - PrivateSubnet1CIDR
          - PrivateSubnet2CIDR

    ParameterLabels:
      VpcName:
        default: "VPC Name"
      VpcCIDR:
        default: "VPC CIDR block"
      PublicSubnet1CIDR:
        default: "Public Subnet 1"
      PublicSubnet2CIDR:
        default: "Public Subnet 2"
      PrivateSubnet1CIDR:
        default: "Private Subnet 1"
      PrivateSubnet2CIDR:
        default: "Private Subnet 2"

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VpcCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: !Ref VpcName

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: !Ref VpcName

  InternetGatewayAttachment:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      InternetGatewayId: !Ref InternetGateway
      VpcId: !Ref VPC

  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs '' ]
      CidrBlock: !Ref PublicSubnet1CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub
            - ${VpcName} Public Subnet (${AZ1})
            - { AZ1: !Select [ 0, !GetAZs  '' ] }
  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 1, !GetAZs  '' ]
      CidrBlock: !Ref PublicSubnet2CIDR
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: !Sub
            - ${VpcName} Public Subnet (${AZ2})
            - { AZ2: !Select [ 1, !GetAZs  '' ] }
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 0, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnet1CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub
            - ${VpcName} Private Subnet (${AZ1})
            - { AZ1: !Select [ 0, !GetAZs  '' ] }
  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      AvailabilityZone: !Select [ 1, !GetAZs  '' ]
      CidrBlock: !Ref PrivateSubnet2CIDR
      MapPublicIpOnLaunch: false
      Tags:
        - Key: Name
          Value: !Sub
            - ${VpcName} Private Subnet (${AZ2})
            - { AZ2: !Select [ 1, !GetAZs  '' ] }

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${VpcName} Public Routes

  DefaultPublicRoute:
    Type: AWS::EC2::Route
    DependsOn: InternetGatewayAttachment
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnet1RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet1

  PublicSubnet2RouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId: !Ref PublicRouteTable
      SubnetId: !Ref PublicSubnet2

  PrivateRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: !Sub ${VpcName} Private Routes

  DatabaseSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Udacity ARR Project - Database Security Group
      GroupName: UDARR-Database
      SecurityGroupIngress:
        -   Description: Application EC2 instances
            FromPort: 3306
            IpProtocol: tcp
            SourceSecurityGroupId: !Ref ApplicationSecurityGroup
            ToPort: 3306
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: UDARR-Database

  ApplicationSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Udacity ARR Project - Application Security Group
      GroupName: UDARR-Application
      SecurityGroupIngress:
        -   Description: SSH from the Internet
            FromPort: 22
            IpProtocol: tcp
            CidrIp: 0.0.0.0/0
            ToPort: 22
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: UDARR-Application

Outputs:
  VPC:
    Description: VPC ID
    Value: !Ref VPC

  PublicSubnets:
    Description: A list of the public subnets
    Value: !Join [ ", ", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]

  PrivateSubnets:
    Description: A list of the private subnets
    Value: !Join [ ", ", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]

  DatabaseSecurityGroup:
    Description: ID of the database security group
    Value: !Ref DatabaseSecurityGroup

  ApplicationSecurityGroup:
    Description: ID of the EC2 instance security group
    Value: !Ref ApplicationSecurityGroup
```


* Highly durable RDS Database:
1. Create a new RDS Subnet group in the active and standby region using private subnets.
2. Create a new MySQL, multi-AZ database in the active region

![alt text](https://github.com/mikethwolff/AWS-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/primary-DB-subnetgroups.png)

![alt text](https://github.com/mikethwolff/AWS-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/primary-DB-config.png)


* Availability Estimate
1. Describing the achievable Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for this Multi-AZ, multi-region database.

* Demonstrate normal usage
1. Create an EC2 keypair in the region
2. Launch an Amazon Linux EC2 instance in the active region. Configure the instance to use the VPC's public subnet and security group ("UDARR-Application").
3. SSH to the instance and connect to the "udacity" database in the RDS instance.
4. Verify that you can create a table, insert data, and read data from the database.
5. You have now demonstrated that you can read and write to the primary database

* Monitor database: 
1. Observe the “DB Connections” to the database and how this metric changes as you connect to the database
2. Observe the “Replication” configuration with your multi-region read replica.

**Solution:** [Design for Availability, Reliability, and Resiliency](https://github.com/mikethwolff/AWS-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/README.md))
