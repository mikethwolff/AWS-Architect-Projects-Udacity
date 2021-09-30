# AWS Cloud Architect Projects Udacity

*The listed projects were part of my AWS Cloud Architect Nanodegree and are displayed in derivated form due to possible copyright restrictions.*

*Form and requirements of my projects may differ from present projects. Udacity may vary projects and this repo is representing my findings at this point in time and may not be representative for future  AWS Cloud Architect projects. If you are currently enrolled into Udacity's AWS Cloud Architect Nanodegree please keep in mind that my findings can help you as a guide but copying these findings will go against Udacity's Honor Code.*


# Project 1: Design for Availability, Reliability, and Resiliency

In this project you will create highly available solutions to common use cases. You will build a Multi-AvailabilityZone, Multi-Region database and show how to use it in multiple geographically separate AWS regions. You will also build a website hosting solution that is versioned so that any data destruction and accidents can be quickly and easily undone.

There will be three parts to this project:

**Part 1:**
1. Data Durability And Recovery:
- Pick two AWS regions. An active region and a standby region. 
- Use CloudFormation to create one VPC in each region.

**Solution:**

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


2. Highly durable RDS Database:
- Create a new RDS Subnet group in the active and standby region using private subnets.
- Create a new MySQL, multi-AZ database in the active region

**Solution:**

![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/primary-DB-subnetgroups.png)

![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/primary-DB-config.png)

![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/secondary-DB-subnetgroups.png)

![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/secondary-DB-config.png)

3. Availability Estimate
- Describing the achievable Recovery Time Objective (RTO) and Recovery Point Objective (RPO) for this Multi-AZ, multi-region database.

**Solution:**

estimates.txt
```
AWS offers various disaster recovery options in the cloud. RPO & RTO depend on what architecture you choose.

Disaster recovery strategies available within AWS can be broadly categorized into four approaches, ranging from the low cost and low complexity of making backups, over pilot light setups, warm standby, to most complex strategies using multiple active Regions. 

Backup & Restore: RPO / RTO: hours; lower priority use cases; Restore data after event; Deploy resources after event; low cost
Pilot Light: RPO / RTO: 10s of minutes; les stringent RTO & RPO; Core services; Start & scale resources after event; medium cost
Warm standby: RPO / RTO: Minutes; More stringent RTO & RPO; Business critical services; Scale resources after even; high cost
Multi-site active / active: RPO / RTO: Real-time; Zero downtime; Near zero loss; Mission critical services; highest cost

Source: https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html

1. Minimum RTO for a single AZ outage
-------------------------------------------------
--- 60-120 seconds

The Recovery Time Objective (RTO) is the duration of time and a service level within which a business process must be restored after a disaster in order to avoid unacceptable consequences associated with a break in continuity.

Minimum RTO is the time Amazon needs to automatically switch to a standby replica in another Availability Zone, which takes typically 60-120 seconds.

"... Failover times are typically 60–120 seconds. However, large transactions or a lengthy recovery process can increase failover
time. When the failover is complete, it can take additional time for the RDS console to reflect the new Availability Zone."

Source: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html


2. Minimum RTO for a single region outage
-----------------------------------------------------
--- 30 minutes up to 1 hour

Minimum RTO for starting up a new Amazon RDS instance and applying all changes since the last backup takes 30 minutes up to 1 hour.


3. Minimum RPO for a single AZ outage
-------------------------------------------------
--- 5 minutes

Recovery point objective (RPO) describes a period of time in which an enterprise's operations must be restored following a disruptive event, e.g., a cyberattack, natural disaster or communications failure.

"RPO for recovery with an RDS Single-AZ instance failure is typically 5 minutes (the interval required for copying transaction logs to Amazon S3), but it can vary."


4. Minimum RPO for a single region outage
-----------------------------------------------------
--- vary between 5 minutes and hours

"The RTO timing requires starting up a new Amazon RDS instance and then applying all changes since the last backup. The RPO is typically 5 minutes, but you can find it by calling RDS:describe-db-instances:LatestRestorableTime. This time can vary from 10 minutes to hours, depending on the number of logs that need to be applied. It can only be determined by testing because it depends on the size of the database, the number of changes made since the last backup, and the workload levels on the database. The RDS backups and transaction logs are stored in Amazon S3, so this recovery can occur in any supported Availability Zone in the Region."

Source: https://aws.amazon.com/blogs/database/amazon-rds-under-the-hood-single-az-instance-recovery/
```

4. Demonstrate normal usage
- Create an EC2 keypair in the region
- Launch an Amazon Linux EC2 instance in the active region. Configure the instance to use the VPC's public subnet and security group ("UDARR-Application").
- SSH to the instance and connect to the "udacity" database in the RDS instance.
- Verify that you can create a table, insert data, and read data from the database.
- You have now demonstrated that you can read and write to the primary database

**Solution:**

logfile_primary.txt
```
[root@ip- ec2-user]# mysql -h [ db Endpoint ] -P 3306 -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 131
Server version: 8.0.23 Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> CREATE DATABASE TestingPrimary;
Query OK, 1 rows affected (0.00 sec)

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| TestingPrimary     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| udacity            |
+--------------------+
6 rows in set (0.00 sec)

MySQL [(none)]> use TestingPrimary
Database changed

MySQL [TestingPrimary]> CREATE TABLE IF NOT EXISTS testing (  test_id INT AUTO_INCREMENT,  test_content VARCHAR(255) NOT NULL,  PRIMARY KEY (test_id) );
Query OK, 0 rows affected (0.07 sec)

MySQL [TestingPrimary]> show tables
    -> ;
+--------------------------+
| Tables_in_TestingPrimary |
+--------------------------+
| testing                  |
+--------------------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]> INSERT INTO testing (test_content) VALUES ('Still testing');
Query OK, 1 row affected (0.01 sec)

MySQL [TestingPrimary]> SELECT * FROM testing;
+---------+---------------+
| test_id | test_content  |
+---------+---------------+
|       1 | Still testing |
+---------+---------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]>

```

logfile_secondary_before_promote.txt
```
[root@ip- ec2-user]# mysql -h [ secondary db Endpoint ] -P 3306 -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 119
Server version: 8.0.23 Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| TestingPrimary     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| udacity            |
+--------------------+
6 rows in set (0.01 sec)

MySQL [(none)]> use TestingPrimary
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [TestingPrimary]> show tables;
+--------------------------+
| Tables_in_TestingPrimary |
+--------------------------+
| testing                  |
+--------------------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]> SELECT * FROM testing;
+---------+---------------+
| test_id | test_content  |
+---------+---------------+
|       1 | Still testing |
+---------+---------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]> INSERT INTO testing (test_content) VALUES ('Testing secondary');
ERROR 1290 (HY000): The MySQL server is running with the --read-only option so it cannot execute this statement
MySQL [TestingPrimary]> Connection to 34.207.173.189 closed by remote host.
Connection to 34.207.173.189 closed.
```

logfile_secondary_after_promote.txt
```
[ec2-user@ip- ~]$ mysql -h s[ secondary db Endpoint ] -P 3306 -u admin -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.23 Source distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| TestingPrimary     |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| udacity            |
+--------------------+
6 rows in set (0.00 sec)

MySQL [(none)]> use TestingPrimary
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MySQL [TestingPrimary]> show tables;
+--------------------------+
| Tables_in_TestingPrimary |
+--------------------------+
| testing                  |
+--------------------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]> SELECT * FROM testing;
+---------+---------------+
| test_id | test_content  |
+---------+---------------+
|       1 | Still testing |
+---------+---------------+
1 row in set (0.00 sec)

MySQL [TestingPrimary]> INSERT INTO testing (test_content) VALUES ('Testing secondary');
Query OK, 1 row affected (0.01 sec)

MySQL [TestingPrimary]> SELECT * FROM testing;
+---------+-------------------+
| test_id | test_content      |
+---------+-------------------+
|       1 | Still testing     |
|       2 | Testing secondary |
+---------+-------------------+
2 rows in set (0.00 sec)

MySQL [TestingPrimary]>
```

5. Monitor database: 
- Observe the “DB Connections” to the database and how this metric changes as you connect to the database
- Observe the “Replication” configuration with your multi-region read replica.

**Solution:**

![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/monitoring_connections.png)
![alt text](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/screenshots/monitoring_replication.png)

**Link to project 1 solution:** [Design for Availability, Reliability, and Resiliency](https://github.com/mikethwolff/AWS-Cloud-Architect-Projects-Udacity/blob/main/Design%20for%20Availability%2C%20Reliability%2C%20and%20Resiliency/README.md))
