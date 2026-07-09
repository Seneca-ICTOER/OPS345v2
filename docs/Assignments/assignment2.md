---
id: assignment2
title: Assignment 2
sidebar_position: 2
description: OPS345 Assignment 2
---

# Assignment 2: Elastic Load Balancing and Auto Scaling

## Objectives of Assignment 2

In this assignment, you are going to configure **Auto Scaling** and **Elastic Load Balancing** for your **Elastic Beanstalk** application. This will build on the Wordpress deployment using _Elastic Beanstalk_ from Lab 8. These services allow you to take an application you have built and turn it into a high capacity available application: meaning users will not see any degradation in performance regardless of how many users are connected. In the case of your **Wordpress** site, you will not have a high number of users trying to access it at any given time. However, the concepts of auto scaling and load balancing are part of what has lead to the popularity of the cloud. Load balancing can also be done (on premises) using things like a **hardware load balancer** or by installing and configuring a service such as **HA Proxy** (on a unique server).

AWS auto scaling requires some configuration options, such as:

- **Metric**: Metric used for your Auto Scaling trigger.
- **Statistic**: Statistic calculation the trigger should use, such as Average.
- **Unit**: Unit for the trigger metric, such as Bytes.
- **Period**: Specifies how frequently Amazon CloudWatch measures the metrics for your trigger.
- **Breach duration**: Amount of time, in minutes, a metric can be outside of the upper and lower thresholds before triggering a scaling operation.
- **Upper threshold**: If the metric exceeds this number for the breach duration, a scaling operation is triggered.
- **Scale up increment**: The number of Amazon EC2 instances to add when performing a scaling activity.
- **Lower threshold**: If the metric falls below this number for the breach duration, a scaling operation is triggered.
- **Scale down increment**: The number of Amazon EC2 instances to remove when performing a scaling activity.

Navigate to [Elastic Beanstalk Console](https://console.aws.amazon.com/elasticbeanstalk) and click on your Environment (**Wordpress-env**).

1. In the navigation pane (left side), choose Configuration.
1. Scroll down to **Instance traffic and scaling**, and click **Edit**.
1. Under **Capacity** > **Auto Scaling Group** select **Load balanced** from the **Environment type** dropdown.
1. Under **Instances** set the following:
    - Min: **2**
    - Max: **3**
1. In the Scaling triggers section, configure the following settings:
    - Metric: **RequestCount**
    - Unit: **Count**
    - Period: **2 Min**
    - Breach duration: **1 Min**
    - Upper threshold: **5**
    - Scale up increment: **1**
    - Lower threshold: **1**
    - Scale down increment: **-1**
1. Under **Load balancer subnets**, make sure **ONLY** the following are checked:

| Availability Zone | Name |
| :--------- | :--------- | 
| **us-east-1a** | **Public Subnet 1** |
| **us-east-1b** | **Public Subnet 2** |

To save the changes choose **Apply** at the bottom of the page. It will take a few minutes for **Elastic Beanstalk** to update the environment.

Navigate to your Wordpress main page by clicking on the link provided in the Elastic Beanstalk **Environment Overview**.

Login to [My.Custom.Domain](https://mycustomdomain.senecapolytechnic.ca/). Repeat the steps from lab 4 to add a record, with the following information:
- Name: **a2**
- Type: **CNAME Record**
- Value: **Your elastic beanstalk application URL** ie **wordpress-env.eba-pehkxxfr.us-east-1.elasticbeanstalk.com**
- Course: **OPS345**
- Description: **CNAME record for Elastic Beanstalk Wordpress Application**

Try navigating to **a2.username.mystudentproject.ca** where **username** is your username. You should see your wordpress site. You may need to wait a few minutes for the CNAME record to propagate. You can use the [Google Admin ToolBox](https://toolbox.googleapps.com/apps/dig/#A/) to verify it is working.

### Blog Posts:

Add a blog post detailing the following. You may need to do a little light research.

- What is **Load Balancing**?
- What is **Horizontal Scaling** aka **Scaling Out**?
- What is **Vertical Scaling** aka **Scaling Up**?
- What is **Auto Scaling**?

Add a second blog post detailing the following:
- What did you think of this assignment?
- What was the most difficult part for you?
- What was the easiest part for you?
- What would you change about this course?

## Submission
Submit screenshots showing:
- The two instances created via Auto Scaling in EC2 (navigate to **EC2** > **Instances**)
- Your blog posts
- Your CNAME record

## Rubric
Assignment 2 is worth 10% of your Final Grade. Your submission will be graded according to the following criteria:

| Task                                          | Mark   |
| :-------------------------------------------- | :----- |
| Two instances in EC2                          | 8      |
| Wordpress showing domain name                 | 6      |
| First Blog Post                               | 3      |
| Second Blog Post                              | 3      |
| **Total**                                     | **20** |
