# AWS Cloud Architect Projects Udacity

The listed projects were part of my AWS Cloud Architect Nanodegree and are displayed in derivated form due to copyright restrictions.

Form and requirements of my projects may differ from present projects. Udacity may vary projects and this repo is representing my findings at this point in time and may not be representative for future  AWS Cloud Architect projects. If you are currently enrolled into Udacity's AWS Cloud Architect Nanodegree please keep in mind that my findings can help you as a guide but copying these findings will go against [Udacity Honor Code](https://udacity.zendesk.com/hc/en-us/articles/210667103-Udacity-Honor-Code).


## Project 1: Design for Availability, Reliability, and Resiliency

In this project you will create highly available solutions to common use cases. You will build a Multi-AvailabilityZone, Multi-Region database and show how to use it in multiple geographically separate AWS regions. You will also build a website hosting solution that is versioned so that any data destruction and accidents can be quickly and easily undone.

There will be three parts to this project:

**Part 1:**
* Data Durability And Recovery:
1. Pick two AWS regions. An active region and a standby region. 
2. Use CloudFormation to create one VPC in each region.

* Highly durable RDS Database:
1. Create a new RDS Subnet group in the active and standby region using private subnets.
2. Create a new MySQL, multi-AZ database in the active region

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
