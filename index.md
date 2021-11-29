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

This guide will be focusing on one of the more expensive instance types, the G4dn (NVIDIA T4) instance type due to its high hourly consumption rate but it can apply to any instance type as well. Also, Teradici CAS agent software does have an optional [idle resource management service](https://www.teradici.com/web-help/cas_manager_as_a_service/reference/install_configure_cam_idle_shutdown/) that requires additional registry configuration on each workstation to achieve a similar result. This proposed solution is region wide, automatically applied on instance start-up and doesn’t require and configuration on the host system.

**AWS EC2 PCoIP Quick connect script workflow, components and dependencies.**

![image](https://github.com/ChadSmithTeradici/AWS_EC2_Stop_Idle_Instance_Lambda_CloudWatch/blob/main/images/StopLongRunninginstances.jpg)
