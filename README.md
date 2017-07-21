# Deploying Microservices with Amazon ECS, AWS CloudFormation, and an Application Load Balancer

This repository (based on the reference architecture located [here](https://github.com/awslabs/ecs-refarch-cloudformation)) provides a set of YAML templates for deploying microservices to [Amazon EC2 Container Service (Amazon ECS)](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) with [AWS CloudFormation](https://aws.amazon.com/cloudformation/).

## TL;DR

If you're in a hurry:
 - First, deploy an VPC for your environment (prod, dev, staging, etc) using [this template](infrastructure/vpc.yaml) [![cloudformation-launch-stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=vpc-testing&templateURL=https://s3.amazonaws.com/cfn-truckpad/ecs/infrastructure/vpc.yaml). Fill the **Environment Name** field with the same value as the **Stack Name** field.
 - Then, deploy an ECS cluster that uses the VPC that you've created. Type the VPC **Environment Name** into the **Existing VPC to attach** field (another VPC will be created if this field is empty). If you want to use On Demand instances, use [this template](master-normal.yaml) [![cloudformation-launch-stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=ecs-testing-normal&templateURL=https://s3.amazonaws.com/cfn-truckpad/ecs/master-normal.yaml). If you want to use Spot Instances, use [this template](master-spot.yaml) [![cloudformation-launch-stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?#/stacks/new?stackName=ecs-testing-spot&templateURL=https://s3.amazonaws.com/cfn-truckpad/ecs/master-spot.yaml).
 - You can have more than one ECS cluster within each VPC that you create - one on-demand cluster for web-based containers and another spot cluster for workers is nice!
 - That's it! :D

## Overview

The repository consists of a set of nested templates that deploy the following:

 - A tiered [VPC](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Introduction.html) with public and private subnets, spanning an AWS region.
 - A highly available ECS cluster deployed across two [Availability Zones](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) in an [Auto Scaling](https://aws.amazon.com/autoscaling/) group.
 - A pair of [NAT gateways](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-nat-gateway.html) (one in each zone) to handle outbound traffic.
 - Two interconnecting microservices deployed as [ECS services](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) (website-service and product-service). 
 - An [Application Load Balancer (ALB)](https://aws.amazon.com/elasticloadbalancing/applicationloadbalancer/) to the public subnets to handle inbound traffic.
 - ALB path-based routes for each ECS service to route the inbound traffic to the correct service.
 - Centralized container logging with [Amazon CloudWatch Logs](http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/WhatIsCloudWatchLogs.html).

## Template details

The templates below are included in this repository and reference architecture:

| Template | Description |
| --- | --- | 
| [master-normal.yaml](master-normal.yaml) | This is the master template for an ECS cluster with On Demand instances - deploy it to CloudFormation and it includes all of the others automatically. |
| [master-spot.yaml](master-spot.yaml) | This is the master template for an ECS cluster with Spot instances - deploy it to CloudFormation and it includes all of the others automatically. |
| [infrastructure/vpc.yaml](infrastructure/vpc.yaml) | This template deploys a VPC with a pair of public and private subnets spread across two Availability Zones. It deploys an [Internet gateway](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Internet_Gateway.html), with a default route on the public subnets. It deploys a pair of NAT gateways (one in each zone), and default routes for them in the private subnets. |
| [infrastructure/security-groups.yaml](infrastructure/security-groups.yaml) | This template contains the [security groups](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_SecurityGroups.html) required by the entire stack. They are created in a separate nested template, so that they can be referenced by all of the other nested templates. |
| [infrastructure/load-balancers.yaml](infrastructure/load-balancers.yaml) | This template deploys an ALB to the public subnets, which exposes the various ECS services. It is created in in a separate nested template, so that it can be referenced by all of the other nested templates and so that the various ECS services can register with it. |
| [infrastructure/ecs-cluster.yaml](infrastructure/ecs-cluster.yaml) | This template deploys an ECS cluster to the private subnets using an Auto Scaling group. |
| [services/dd-agent/service.yaml](services/dd-agent/service.yaml) | This is an long-running ECS service that monitors your cluster and apps with Datadog.|

After the CloudFormation templates have been deployed, the [stack outputs](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/outputs-section-structure.html) contain a link to the load-balanced URLs for each of the deployed microservices.


## License

Copyright 2017 TruckPad. All Rights Reserved.

Copyright 2011-2016 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License"). You may not use this file except in compliance with the License. A copy of the License is located at

[http://aws.amazon.com/apache2.0/](http://aws.amazon.com/apache2.0/)

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

