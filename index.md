---
title: Quick connect to instances in AWS with a PCoIP session
description: Script that powers-up Instances, identified external IPs and establishes a PCoIP connection in a single step.
author: chad-m-smith
tags: AWS, Active Directory, EC2
date_published: 2021-11-05
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

This script is designed to allow Teradici end users to power on their EC2 instances remotely without having to access the EC2 dashboard. It will also find the public or elastic IP of the instances and pass the connection info to a PCoIP connection string automatically. While larger Teradici deployments will benefit from Cloud Access Manager (CASM) when multiple instances reside in the same region. It is cost prohibited to run CASM connection gateways (CAC) when only a handful of user’s instances per region are needed and/or customer are interested in leveraging AWS local zones where the cost of perpetually running a CAC is cost prohibited!

## Objectives

+ Create EC2 instances in AWS.
+ Create an IAM role/policy to lock down access to instances
+ Apply Policy to user and programmatic access to resources 
+ Download and configure AWS CLI 
+ Create script and set permission on client.


## Costs

This tutorial uses billable components of AWS Cloud and assumes Teradici subscription, including the following:
+   [AWS Nvidia EC2 Instance](https://aws.amazon.com/nvidia/), including vCPUs, memory, disk, and GPUs
+   [Internet egress and transfer costs](https://aws.amazon.com/blogs/architecture/overview-of-data-transfer-costs-for-common-architectures/), for PCoIP and other applications communications.

Use the [AWS pricing calculator](https://calculator.aws/#/) to generate a cost estimate based on your projected usage.

## Before you begin

In this section, you set up some basic resources that the tutorial depends on.

1. Instructions in this guide assume that you have a [AWS account](https://aws.amazon.com/free/) 

1. Familiarize yourself with [AWS CLI](https://aws.amazon.com/cli/)

1. Understand [IAM roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) 

## Creation of PCoIP EC2 Instances
In this section, you procure will procure a EC2 instance through the EC2 Dashboard. This section isn't an exhaustive explanation instead rather focusing on EC2 power on script. For more details directions on the actual installation process there are two deployment methodologies; [AWS marketplace](https://aws.amazon.com/marketplace/search/results?searchTerms=teradici) (or), refer to [EC2 Nvidia](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Ultra-deployment-script-for-AWS-NVIDIA-EC2-instances) and [EC2 standard](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Standard-deployment-script-for-AWS-EC2-instances) installation guides for customer wanting to bring in their existing Teradici registration codes to AWS.

## Configure a IAM access policy for EC2 instances
Create an IAM policy for EC2 instance to read the secrets through the installation script to join the domain.

1. Go to IAM -> Policy -> Create Policy. 
    
1. Select the **Create Policy** button

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/Create_Policy_Button.jpeg)
    
Select the **JSON** tab, it will open in separate web browser tab. Click on tab JSON and paste the following text. Also add in the **ARN** for the **EC2 Instances** captured in the previous step when creating or assigning the EC2 instances. 

**Note** The permission in this IAM policy are pretty broad for the EC2 instance and your organization may want to further lock-down this IAM policy further.

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:RunInstances",
                "ec2:TerminateInstances",
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": [
                "arn:aws:ec2:us-west-2:000000000000:instance/i-00000000000000001",
                "arn:aws:ec2:us-west-1:455311239824:instance/i-00000000000000002"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceStatus",
                "ec2:DescribeAddresses",
                "ec2:AssociateAddress",
                "ec2:DisassociateAddress",
                "ec2:DescribeRegions",
                "ec2:DescribeAvailabilityZones"
            ],
            "Resource": "*"
        }
    ]
}
```
3. Select Option tag then **Next** to continue

4. Review the setting to the IAM role, **Name** the Policy then select the **Create Policy button** to finish creating the role.

## Creation of a IAM User to access policys resources
Within the IAM Management Console, select the Create **user** option.

1. In the IAM User section select **Add User** button.

1. In the User creation section, you will assign an **name** and check-box for **programmatic access** to resources. Click **Next:Permissions**

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/Add_users.jpg)
    
1. In the Set Permission section, you can decided on what security and access methodology your organization needs. Are you going to grant EC2 access on a per user basis,then **Attach existing policy directly** option is the best. If you are going to provide access to instances to a group of users, then **Add user to group** is advised. For this guide we will assoicating these instances directly a individual user.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/Set_permissions.jpg)
    
1. Next add optional **Tags**, Select **Next** to continue.

1. Finally, review the settings for the user and ensure finish by selecting the **Create User**.

1. The next window will show the credentials for programmatic access (Access and Security keys) that will be required later when configuring AWS CLI access. Copy or download the access key and secuity in a secure location for future reference. You only have one time to gain access to the security key, without having to generate a new pair.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/Programmatic_access.jpg)

## Installation of AWS CLI on client

## Creation of power-on script per OS

## Executing the Script as shortcut

## Revoke access to EC2 Instance

It is stronly recommended to remove access to the AWS secrets after you have succefully leveraged the domain join script. A savy end-user could theoretically access the AWS secrets while logged into an instance that still has granted access to the IAM role and they are aware ofthe Secrets Name to call in a function.  

**Option 1:** Apply an explicite Deny to the IAM role that overrides the IAM Policy orginally created**
1. From the [IAM Dashboard](https://console.aws.amazon.com/iam), select the **Role** option in the left pane. 

1. Search for the name of the Role that was previously created. *(Example: Domain_Join_script)*

1. Select the **Revoke** tab with the Role properties. 
    
![image](https://github.com/ChadSmithTeradici/DomainJoin-with-AWS-Secrets-for-Windows-EC2-instances/blob/main/images/Revoke_IAM_Role.jpg)

**Note:** An explicit DENY policy will override any ALLOW policies assinged to the same role. If you wanted to re-run the Domain Join script again and access the secrets, you would have to remove the DENY policy first.
    
**Option 2:** Remove the IAM Role on a per-instance basis through the EC2 Dashboard.

1. From [EC2 Dashboard](https://console.aws.amazon.com/ec2), select the instances by **checking the box** left of the instances that just have been added to AD.

1. Select the **Actions** button, the **Security**, **Modify IAM Role**

1. In the Modify IAM Role window, select the **No IAM Role** option to remove access to secrets. 

![image](https://github.com/ChadSmithTeradici/DomainJoin-with-AWS-Secrets-for-Windows-EC2-instances/blob/main/images/Remove_IAM_Role_Instance.jpg)

![image](https://github.com/ChadSmithTeradici/DomainJoin-with-AWS-Secrets-for-Windows-EC2-instances/blob/main/images/Apply_No_IAM_Role.jpg)
