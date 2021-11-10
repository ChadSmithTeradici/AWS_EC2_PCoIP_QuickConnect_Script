---
title: Quick connect to instances in AWS with a PCoIP session
description: Script that powers-up Instances, identified external IPs and establishes a PCoIP connection in a single step.
author: chad-m-smith
tags: AWS, EC2, Power Management
date_published: 2021-11-05
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

This script is designed to allow Teradici end users to power on their EC2 instances remotely without having to access the EC2 dashboard. It will also find the public or elastic IP of the instances and pass the connection info to a PCoIP connection string automatically. While larger Teradici deployments will benefit from [Cloud Access Manager (CASM)](https://www.teradici.com/web-help/cas_manager_as_a_service/) when multiple instances reside in the same region. It is cost prohibited to run CASM connection gateways (CAC) when only a handful of user’s instances per region are needed and/or customer are interested in leveraging AWS local zones where the cost of perpetually running a CAC is cost prohibited.

**AWS EC2 Quick connect script workflow, components and dependencies.**

   ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/AWSAWS_EC2_PCoIP_QuickConnect_Script.jpg)

## Objectives

+ Create EC2 instances in AWS.
+ Create an IAM role/policy to lock down access to instances
+ Apply Policy to user and programmatic access to resources 
+ Download and configure AWS CLI 
+ Install PCoIP Client software
+ Create script and set permission on client.


## Costs

This tutorial uses billable components of AWS Cloud and assumes Teradici subscription, including the following:
+   [AWS EC2 Instance](https://aws.amazon.com/pm/ec2/), including vCPUs, memory, disk, and GPUs
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
Create an [IAM policy](https://console.aws.amazon.com/iamv2/home#/policies) for EC2 instance to read the secrets through the installation script to join the domain.

1. Go to IAM -> Policy -> Create Policy. 
    
1. Select the **Create Policy** button

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_PCoIP_QuickConnect_Script/blob/main/images/Create_Policy_Button.jpeg)
    
Select the **JSON** tab, it will open in separate web browser tab. Click on tab JSON and paste the following text. Also add in the **ARN** for the **EC2 Instances** captured in the previous step when creating or assigning the EC2 instances to the user.

**Note:** The permission in this IAM policy are pretty broad for the EC2 instance and your organization may want to further lock-down this IAM policy further.

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
                "arn:aws:ec2:us-west-2:000000000001:instance/i-00000000000000001",
                "arn:aws:ec2:us-west-1:000000000001:instance/i-00000000000000002"
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
Within the [IAM Management Console](https://console.aws.amazon.com/iamv2/home#/home), select the Create **user** option.

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
1. Installation of [AWS CLI](https://aws.amazon.com/cli/) client based on client OS
1. [Configure AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) with provisioned Access key and Secret Key
   In a terminal session (or) cmd window, type in *aws configure* and fill in the **Access / Security keys** provided when creating auser, as well as the **Default      regions name** and the **Default output format**. 
   
   An example aws configure command:
  
   ```
    $ aws configure
    AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
    AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    Default region name [None]: us-west-2
    Default output format [None]: json
   ```
## Installation of CAS client

Install [PCoIP Software Clients](https://docs.teradici.com/find/product/software-and-mobile-clients) based on your desired OS. 
You are free to the install PCoIP client on a many devices as you wish. 
 
## Creation of power-on / connect script per OS

After the installation of AWS CLI and programmatic access has been assigned to the client. You can now copy either of the below scripts based on your OS and change the permissions in order to execute.

The script logic is to have a *instance-id* and its assoicated *region* pre set as a default, that can be 'clicked thru' to quickly establish a PCoIP connection. The script is designed to allow users to enter in a different instance-id and it's assoicated region if the user needs access a different instances. As long as the assoicated IAM policy has granted access to other instances. *(In the example policy earlier we had two instances 'us-west-2 and us-west-1' available to log into instances.)*

For Windows clients, copy the script below into a text editor and modify the 2nd line replaceing *i-00000000000001* with the instance-ID for the default instance ID as well as the region *(us-west-2)* that instance resides in for the 3rd line.  Save the file with a .ps1 extention and excute as a powershell script. 

```
$cmd = 'powershell.exe'
$defaultInstanceID = 'i-00000000000001'
$defaultRegion = 'us-west-2'
$region = Read-Host "Press enter to accept the default region: [$($defaultRegion)]"
$region = ($defaultRegion,$region)[[bool]$region]
aws configure set default.region $region
$prompt = Read-Host "Press enter to accept the default Instance id: [$($defaultInstanceID)]"
$prompt = ($defaultInstanceID,$prompt)[[bool]$prompt]
aws ec2 start-instances --instance-ids $prompt
Write-Output "Timeout is for 20 seconds for Instance to start in AWS"
timeout /T 20
Write-Output "Will poll EC2 instance for External IP address"
sleep -s 1.5
$ExtIP=aws ec2 describe-instances --instance-ids $prompt --query 'Reservations[*].Instances[*].PublicIpAddress' --output text
Write-Output ("Found External IP Address of " + $ExtIP + " will try a establish a PCoIP connection..")
sleep -s 1.5
$Launch = "pcoip://$ExtIp"
Start-Process $Launch
```
For Linux and Mac clients, copy the script below into a text editor and modify the line 2 for description and line 3, replacing *us-west-2* with the default Instances region. Also, modify line 4 for the instance-id description and line 5, replacing *I-0000000000000001* with the default Instance-ID.

**REMEMBER** to keep the "-" in front for both instance-ID and region in lines 3 and 5, for bash script to work correctly. 

```
#!/bin/bash
read -p "Press enter to accept the default region or select different if desired instance is in different location [us-west-2]: " region
region=${region:-us-west-2}
read -p "Press enter to accept the default Instance id or enter new [I-0000000000000001]: " name
name=${name:-I-0000000000000001}
aws configure set region $region
sleep 1
aws ec2 start-instances --instance-ids $name
echo "Timeout is for 20 seconds for Instance to start in AWS"
sleep 20
echo "Will poll EC2 instance for External IP address"
sleep 1
extIP=$(aws ec2 describe-instances --instance-ids $name --query 'Reservations[*].Instances[*].PublicIpAddress' --output text)
echo "Found External IP Address of " $extIP " will try a establish a PCoIP connection.."
sleep 1
open pcoip://$extIP
```
## Executing the Script and create a shortcut

## Clean up

To avoid incurring charges to your AWS account for the resources used in this tutorial, you can simply delete the instance:

+ In the [EC2 Dashboard](https://console.aws.amazon.com/ec20) go to the EC2 **Instance State** scroll to **Terminate**. 
+ While you don't get charged for [IAM resources](https://console.aws.amazon.com/iamv2/home#/home) you can remove user or remove/modify JSON Policy for users or group as well to tighten security and clean-up policies.

## What's next

+ Configure and optimize the PCoIP expereince on the EC2 Instances for [Windows](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/21.07/admin-guide/configuring/configuring/) or [Linux](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/linux/21.07/admin-guide/configuring/configuring/).
+ Learn more about [Teradici products and offerings](https://www.teradici.com/).
+ Learn more about [AWS EC2 Instances](https://aws.amazon.com/ec2/instance-types/)

