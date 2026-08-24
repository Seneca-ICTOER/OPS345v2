---
id: landing-page
title: Welcome to OPS345
sidebar_position: 1
slug: /
description: Landing Home Page for OPS345
---

# Welcome to OPS345 - Administration of Open Source Systems

## Quick Links

| [Weekly Schedule](./weekly-schedule.md) | [Course Outline](https://apps.senecapolytechnic.ca/ssos/findOutline.do?isLoggedIn=&subjectOrAndTitle=%5BOSL740%5D+Administration+of+Open+Source+Systems&schoolCode=0s867160) | [Assignment 1](/Assignments/assignment1.md) | [Assignment 2](/Assignments/assignment2.md) |
| --------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ------------------------------------------- | 


## What This Course is About

This course teaches the maintenance and administration of Linux servers in both an on-premises and cloud based environment using Amazon Web Services (AWS). Students will learn to install, configure, test and maintain services commonly used by enterprises in a cloud environment. This course is the second in a series of courses about Linux technologies..

- OPS145 taught you to be a Linux user.
- OPS245 taught you to be a Linux system administrator.
- OPS345 will teach you some advanced network/system configuration and how to administer and use cloud-based services for DNS, Web server, databases, containers, and other AWS tools.

In the second half of this course you will use AWS via the AWS learner lab. The AWS Learner Lab provides a sandbox environment where you can build, configure and deploy AWS assets such as instances. There are some limitations as to what you can do, however it provides all the functionality we require for this course.

**You are limited to $50 credit at no cost within the learner lab. Once this runs out, your account will be disabled. Additionally, there is no way to add funds to this pool. We will remind you of this in the second half of the course when we begin with AWS but if you follow the lab instructions properly you should not have any issues.**

## Learning by Doing

Most of the learning in this course occurs through a hands-on approach in the eight labs and two assignments. As such, your Tests and Quizzes will focus on *how* to accomplish something. 

In the first half this will generally mean knowing where in the different operating systems to make changes, what commands to use, and what configuration files to modify and how to modify them. 

In the second half of the course, when the focus switches to AWS, this will mean understanding the network layout in AWS, how to connect the different aspects of an AWS based virtual network, how to control data flow, and how to make different services work together.

However, you will also be expected to understand and will also be tested on *why* things are done that way.

It is a good idea to take well organized notes during the labs and assignments. You will be allowed a reference sheet for your tests and it will be easier to make that reference sheet if you already have well organized notes to pull from. 

## How to run this course

The first half of this course will be run primarily in a set of VMs run inside an Ubuntu operating system. This Ubuntu system will be installed on a hard drive that you will be plugging into the lab computers to do your work (very similarly to how you did your labs in OPS245). Do not use the Debian host system you used in OPS245. A fresh install of Ubuntu is required.

The second half of the class will take place primarily in the AWS learning lab console which is accessed in a web browser and via an SSH connection in your Ubuntu terminal. Use your Ubuntu system for everything you do in the second half. This will prevent possible security issues (such as lost SSH keys) and make it easier for you to make changes to configuration files that you will be modfying in later labs.

**Do not try to complete any of this course in MS Windows. It will cause you unnecessary headaches.**

## Using your own laptop

Some of you may have completed OPS245 using your own laptop. Generally speaking, if you were able to complete OPS245 that way without any trouble, you should be able to do so in OPS345 as well. Again, do not use the same Debian host system that you used in OPS245. You will need a fresh install of Ubuntu to properly complete this course. Keep in mind that the CPU and RAM requirements in this course are more demanding than they were in OPS245. A laptop with 16GB of RAM and a CPU comparable to a 12th Gen i7-12700 or better should be fine but anything less and you are probably better off using the lab machines.

You can technically also use your laptop the same way as the lab machines are used - by plugging an external hard drive into them and booting to that drive via your BIOS. However, different laptop manufacturers treat their default BIOS settings differently and you may need to do some tinkering to get this working. But beware... messing with the BIOS can be dangerous and if you care about the default Windows OS that came with your laptop, you will want to be extra cautious, lest you accidentally break it.

Ultimately this course has been designed and tested to work with the setup we have in the labs - by plugging an external hard drive into the lab machine, installing the Ubuntu host OS onto that hard drive, and booting to it through the lab machine BIOS menu. Using any other method is at the student's discretion. The teacher will not be able to help you with your own laptop.

YOU HAVE BEEN WARNED!

## Reference Sheet Policy

You are allowed a refence sheet for both the Midterm Test and Final Test in this course.

These reference sheets **MUST BE HAND WRITTEN**. You are allowed a one-sided sheet for the Midterm Test. You are allowed a two-sided sheet for the Final Test.

Digital reference sheets will **NOT** be allowed.

## Evaluation

| **Evaluation** | **Marks** |
| -------------- | --------- |
| Labs           | 16%       |
| Assignment (2) | 30%       |
| Quizzes        | 4%        |
| Midterm Test   | 25%       |
| Final Test     | 25%       |
