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

A brief review of the workflow:
1. G4dn instance type is created by end-user or programmatically created in a region where workflow is deployed (regardless of AZ, subnet)
1. A defined Lambda EC2 event trigger assoicaed to a Lamdba function (λ) will execute.
1. If the Instance type is in the G4dn instance family, it will create a CloudWatch event assoicated to the EC2 instances-id.
1. CloudWatch will continue to ‘watch’ the instance and poll the instances CPU utilization every 5 minutes 
1. If the CloudWatch detect  < 5% utilization three time in 15 minutes peroid, it will shut down the instance.
1. If one or more of those periods are above 5% it will continue to poll every 15 minutes interval. 

## Configuring EC2 G4dn idle resource shutdown workflow ##

## Monitoring EC2 Gd4n idle resource shutdown workflow ##

## Disabling EC2 Gd4n idle rerourse Shut down workflow ##



