---
title: Quick connect to instances in AWS with a PCoIP session
description: Script that powers-up Instances, identified external IPs and establishes a PCoIP connection in a single step.
author: chad-m-smith
tags: AWS, EC2, PCoIP, Power Management
date_published: 2021-11-11
---

Chad Smith | Technical Alliance Architect at Teradici | HP

<p style="background-color:#CAFACA;"><i>Contributed by Teradici employees.</i></p>

This script is designed to allow Teradici end users to power on their EC2 instances remotely without having to access the EC2 dashboard. It will also find the public or elastic IP of the instances and pass the connection info to a PCoIP connection string automatically. While larger Teradici deployments will benefit from [Cloud Access Manager (CASM)](https://www.teradici.com/web-help/cas_manager_as_a_service/) when multiple instances reside in the same region. It is cost prohibited to run CASM connection gateways (CAC) when only a handful of user’s instances per region are needed and/or customer are interested in leveraging AWS local zones where the cost of perpetually running a CAC is cost prohibited.

**AWS EC2 PCoIP Quick connect script workflow, components and dependencies.**
