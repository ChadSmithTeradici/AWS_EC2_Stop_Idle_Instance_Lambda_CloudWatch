---
title: Control EC2 cost by shutting down idle instances
description: Monitor and shutdown G4dn types instances per region with AWS Lamdba and CloudWatch Services
author: chad-m-smith
tags: AWS, EC2, Lamdba, CloudWatch, Power Management
date_published: 2021-11-29
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

Controlling costs are important component in adopting cloud. It’s difficult to balance giving your remote creators to access to their resources but at ensuring that there aren’t any ‘runaway costs’ associated to their freedom of work remotely.  

This guide will be focusing on one of the more expensive instance types, the G4dn (NVIDIA T4) due to its high hourly consumption rate but Lamdba function can be applied to any instance type as well. Finally, Teradici CAS agent software does have an optional [idle resource shutdown service](https://www.teradici.com/web-help/cas_manager_as_a_service/reference/install_configure_cam_idle_shutdown/) that requires additional registry configuration on each workstation to achieve a similar result. The reality is that setting up this service diffucult to set-up and manys times not configured on the host systems. This proposed solution is enforced region wide, automatically applied on instance start-up and doesn’t require any configuration on the host systems.

**AWS EC2 G4dn idle resource shutdown workflow, components and dependencies.**

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/StopLongRunninginstances.jpg)

**A brief description of each step in workflow:**
1. G4dn instance type is created by end-user or programmatically created in a region where workflow is deployed (regardless of AZ, subnet)
1. A defined Lambda EC2 event trigger assoicaed to a Lamdba function (λ) will execute.
1. If the Instance type is in the G4dn instance family, it will create a CloudWatch event assoicated to the EC2 instances-id.
1. CloudWatch will continue to ‘watch’ the instance and poll the instances CPU utilization every 5 minutes 
1. If the CloudWatch detect  < 5% utilization three time in 15 minutes peroid, it will shut down the instance.
1. If one or more of those periods are > 5% it will continue to poll every 15 minutes interval. 

# Configuring EC2 G4dn idle resource shut down workflow #
## Create an IAM Role ##
Create an [IAM role](https://console.aws.amazon.com/iamv2/home?#/roles) to provide the proper permissions for the end-user to start and connect to defined instances.

1. Go to IAM -> Role -> Create Role. 
    
1. Select the **Create role** button, select the **Lamdba**

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateRole.jpg)
    
Select the **Next Permissions** button.

1. In Filter policies type in 'EC2Full' and Select the **AmazonEC2FullAccess**

    **Note:** Selecting Full access isn't recommended for most production environment, consult with your security team on narrowing down permission that this           service will run.
    
    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AttachPermission.jpg)    
    
    Select **Next:Tags**
  
1. After *Option* Tag section, The role review page has you name the role. In this example it was named *Lamdba_EC2_long_running_instance*. 

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateRoleReview.jpg)

## Create an Lamdba function ##
Next we will create a Lambda function. In the Lambda function, the function does two things:
- Get the EC2 instance ID set focus on only G4dn instance types
- Create an alarm and attach it to the EC2 instance

1. Select a AWS region that has EC2 Instance type available, the Lamdba function is designed to run in the same region. If you have other instances in other regions / loca zones, you should duplicate this setup as well. Enter the [Lamda dashboard](console.aws.amazon.com/lambda/) and select the **Create function** button

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CreateFunction.jpg)

1. Within the create function wizard, select the **Author from scratch** option.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AuthorFromScatch.jpg)

1. Fill in the basic Lamdba funcation information:
- Our example we named the Lamdba funcation **Stop_idle_EC2-G4dn_instance**
- The Runtime field enter **Python 3.6**
- Select Architecture type as: **x86_64**
- Under Permission > Change default execution role > **Use existing role** > Choose the previously created IAM role above

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/createFuncationBasic.jpg)





## Create CloudWatch events ##
# Monitoring EC2 Gd4n idle resource shut down workflow #

# Disabling EC2 Gd4n idle rerource shut down workflow #



