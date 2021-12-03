---
title: Control EC2 cost by shutting down idle instances
description: Monitor and shutdown G4dn types instances per region with AWS Lamdba and CloudWatch Services
author: chad-m-smith
tags: AWS, EC2, Lamdba, CloudWatch, Power Management
date_published: 2021-11-29
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

Controlling costs are important component in adopting cloud. It’s difficult to balance giving your remote creators to access to their resources anytime of the day all while ensuring that there aren’t any ‘runaway costs’ of idle resources associated to their freedom of working remotely.  

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
- Once the required fields are filled out, Select the **Create funcation** button to continue.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/createFuncationBasic.jpg)

4. Within the funcation overview dashboard, we will start by adding the Lamdba event **trigger** 
- In the 'Select a trigger' box, select the **EventBridge (CloudWatch Events)** type
- Under **Rule**, pick the **Create a new rule** option
- Enter a **Rule name** to identify your rule, example : **Stop_idle_EC2-G4dn_Instances**
- Under **Rule type**, select **Event pattern**
- Use 'drop down box' to select **EC2**
- Use second 'drop down box'to select **EC2 instance state-change notification**
- 'Click' **Detail** to expand options
- Ensure that the **State** box is 'checked'
- Within the **State** field, auto-complete the word **running**
- Once complete select the **Add** on botton of page

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AddTrigger1.jpg)

When successfully completed, a notification will appear says that all subsystem between services have been established.

   ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/SuccessfulTrigger.jpg)

5. Finally, drop in the Python code into the **Code Source** section of the Lamdba Dashboard. Ensure you are in the **Code** section of console.

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CodeToolBar.jpg)

- Remove any pre-populated python code
- **Copy** the snippet of python code below
- Ensure that **line 9** *AlarmActions* has the correct AWS region setup (Example: this code is focused on us-west-2),change if needed.
``` AlarmActions       = ['arn:aws:automate:us-west-2:ec2:stop'],```

```
import boto3


def put_cpu_alarm(instance_id):
    cloudWatch   = boto3.client('cloudwatch')
    cloudWatch.put_metric_alarm(
        AlarmName          = f'CPU_ALARM_{instance_id}',
        AlarmDescription   = 'Alarm when server CPU does not exceed 5%',
        AlarmActions       = ['arn:aws:automate:us-west-2:ec2:stop'],
        MetricName         = 'CPUUtilization',
        Namespace          = 'AWS/EC2' ,
        Statistic          = 'Average',
        Dimensions         = [{'Name': 'InstanceId', 'Value': instance_id}],
        Period             = 900,
        EvaluationPeriods  = 3,
        Threshold          = 5,
        ComparisonOperator = 'LessThanOrEqualToThreshold',
        TreatMissingData   = 'notBreaching'
    )


def lambda_handler(event, context):
    instance_id = event['detail']['instance-id']
    ec2 = boto3.resource('ec2')
    instance = ec2.Instance(instance_id)
    if instance.instance_type.startswith('g4dn'):
        put_cpu_alarm(instance_id)
```
6. In the **Code Source** text editor, select the **File** > **Save** option

    ![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/SavedPythonCode.jpg)

7. Finally, you want to deploy the saved code to run as your Lamdba function. To do this select the **Deploy** button ontop of the text editor tool.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/DeployPythonCode.jpg)
   
**Note** there is notifcation status that will change from **Changes not deployed** to **Changes deployed**

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/PythonSuccDeploy.jpg)

# Monitoring EC2 G4dn idle resource shut down workflow #

The EC2 Dashboard shows summary information aboutCloudWatch alarms per instance.

You get some general information about an alarm being assigned to the instance, by verifying that the Lamdba Function did assigned a alarm to the instance with the **1 in alarm** in the alarm status column.

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AlarmSet.jpg)

The alarm status will change to **1/1 in alarm** and change to red front when the threshold criteria has been meet and the instance was shutdown, this is an easy see with the EC2 Dashboard that CloudWatch event triggered a shutdown without leaving the console. 

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/AlarmAction.jpg)

A more detailed view of the CloudWatch alarms are in the [CloudWatch Dashboard](console.aws.amazon.com/cloudwatch/), which prodives more details on the state of the instances. There may be other alarms active, so you should be note the instance-id of the instance associated alarm you are want to look at. EC2 alarms are referenced to by its instance-ids. In the alarm dashboard, the red line represents the 7% threshold, while the blue line represents the number of polls take and their associated CPU utilization.  The status bar under the graph represents alarm stat, green represents that event criteria can't been meet. A red bar represents the threshold has been meet and in this CloudWatch event, means the instance was shutdown. 

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/CloudWatchAlarm.jpg)

# Disabling EC2 G4dn idle rerource shut down workflow #



