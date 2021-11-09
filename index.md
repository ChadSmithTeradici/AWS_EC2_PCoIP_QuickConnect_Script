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

1. Familiarize yourself with [AWS CLI](https://aws.amazon.com/cli/

1. Understand [IAM roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) 

## Creation of PCoIP EC2 Instances
In this section, you procure will procure a EC2 instance through the EC2 Dashboard. This section isn't an exhaustive explanation instead rather focusing on EC2 power on script. For more details directions on the actual installation process there are two deployment methodologies [AWS marketplace](https://aws.amazon.com/marketplace/search/results?searchTerms=teradici) (or), refer to [EC2 Nvidia](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Ultra-deployment-script-for-AWS-NVIDIA-EC2-instances) and [EC2 standard](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Standard-deployment-script-for-AWS-EC2-instances) installation guides for customer wanting to bring in their existing Teradici registration codes to AWS.![image](https://user-images.githubusercontent.com/92746483/140969823-85c352f2-6c45-40ad-abd7-4e5a8806ef9a.png

## Configure a IAM access policy for EC2 instances

## Creation of a IAM Role to access policy

## Procure a EC2 Instance, assign role and use User Defined Script

In this section, you procure will procure a EC2 instance through the EC2 Dashboard. This section isn't an exhaustive explanation instead rather focusing on domain join script portion. For more details directions on the actual installation process, refer to [EC2 Nvidia](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Ultra-deployment-script-for-AWS-NVIDIA-EC2-instances) and [EC2 standard](https://github.com/ChadSmithTeradici/Teradici-PCoIP-Standard-deployment-script-for-AWS-EC2-instances) installation guides.

1.  Launch a EC2 instance, On the [EC2 Dashboard](https://console.aws.amazon.com/ec2), choose **Launch Instance*

1. On the **Choose AMI** page, select the [Windows 2019 Base](https://aws.amazon.com/marketplace/pp/prodview-bd6o47htpbnoe?ref=cns_srchrow) AMI, then press **Select** button.

1. On the **Choose Instance Type** page, chose a instance type and choose **Next: Configure Instance Details**.

1. On the **Configure Instance Details** page, at a minimum fill in **Networking/Subnet/Auto-Assign Public-IP** based on desired Network topology. Take remaining configuration details based your requirements, until you reach the **IAM Roles**, then select the name of the IAM role previous created. 
*(Example: Domain_Join_script)*

    ![image](https://github.com/ChadSmithTeradici/DomainJoin-with-AWS-Secrets-for-Windows-EC2-instances/blob/main/images/IAM_Role_Domain_Join.jpg)

    Scroll down to the **User data** field in the Advanced Details section and copy the script below and modify accordingly.

   ![image](https://github.com/ChadSmithTeradici/Teradici-PCoIP-deployment_script-for-AWS-NVIDIA-Instances/blob/main/images/User_Data_Field.jpg)   
 
    ```
     <powershell>
    # Domain name and the tld. In this example would be teradici.dom
    $domain_name = "teradici".ToUpper()
    $domain_tld = "dom"
    $secrets_manager_secret_id = "Windows/ServiceAccounts/DomainJoin"

    # Make a request to the secret manager
    $secret_manager = Get-SECSecretValue -SecretId $secrets_manager_secret_id

    # Parse the response and convert the Secret String JSON into an object
    $secret = $secret_manager.SecretString | ConvertFrom-Json

    # Construct the domain credentials
    $username = $domain_name.ToUpper() + "\" + $secret.ServiceAccount
    $password = $secret.Password | ConvertTo-SecureString -AsPlainText -Force

    # Set PS credentials
    $credential = New-Object System.Management.Automation.PSCredential($username,$password)

    # Get the Instance ID from the metadata store, we will use this as our computer name during domain registration.
    $instanceID = [System.Net.Dns]::GetHostName()

    # Perform the domain join
    Add-Computer -DomainName "$domain_name.$domain_tld" -Credential $credential -Passthru -Verbose -Force -Restart
    </powershell>
    ```

If you used the same naming conventations throughout this deployment guide, then the only section you have to personalize is the name of your domain and its       assoicated tld. If you deviated the name of the secrets names or any other variables, then you should make the changes above as well  
    
+ Change to your domain name *Example: teradici* 
+ Chanage to your tld name *Examaple: dom*
    
    ```
    $domain_name = "teradici".ToUpper() 
    $domain_tld = "dom"                
    ```
  **Note, AD server( or) service is resolvable and accessable to the local subnet where instance resides, ensure all required ports are available.**
  
  For the remaining configuration details, make any selections you prefer. Then, choose **Next: Add Storage**.

5. On the **Add Storage** page, choose the Size (GiB) cell and increase the volume based on your requirements. Then, choose **Next: Add Tags**.

6. On the **Add Tags page**, optionally add any Key:Value tags to your instance. Then, choose **Next: Configure Security Group**.

7. On the Configure Security Group page, make the following selections:

    + For **Assign a security group**, choose **Create a new security group**.
    + For **Security group name**, type a descriptive name, such as *pcoip ssh rdp*.
    + For **Description**, optionally add a description.
    + For **Type**, choose **SSH**
    + For **Source**, choose **My IP**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **HTTPS**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom UDP Rule**
    + For **Port Range** choose **4172**
    + For **Source**, choose **0.0.0.0/0**
    + Select **Add rule**
    + For **Type**, choose **Custom TCP Rule**
    + For **Port Range** choose **3389**
    + For **Source**, choose **My IP**
    
    Then, choose **Review** and **Launch**.
    
    **Note:** When the domain join script runs, it will AD join the instace using its instance ID as the AD computer name. Logging into the Active Directory Server     and looking for a correlation between instance ID and computer name ensures that the script has successfully run. 
    
## Revoke access to secrets after domain join

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
