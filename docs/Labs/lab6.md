---
id: lab6
title: Lab 6 - Apache, DNS, and SSL
sidebar_position: 6
description: Configure an Apache web server in AWS, give it a FQDN, and give it SSL with Let's Encrypt
---

# Lab 6 - Configuring an Apache web server in AWS with a FQDN and SSL

## Overview

This week's lab will cover the following:

- Managing Linux packages and updating the system.
- Managing Linux services.
- Modifying Virtual Private Cloud (VPC) Security Groups.
- Implementing an elastic IP in AWS.
- Accessing your Web Server using a browser.
- Using a service to register a domain name
- Generating TLS/SSL certificates with Let's Encrypt & Certbot
- Configuring https on your Apache web server

## Managing Linux packages and updating the system

Installing software in Linux requires both an active Internet connection and knowledge of which package management tool to use for your distribution (or distro). Linux software and updates come from special sources hosted on other servers, known as repositories (or repos). All the major Linux distros host their own repos, though anyone can host a repository for a distribution - and many organizations do. Due to the Open Source nature of Linux, certain repos may contain specialized software that is not available in the main repository (such as EPEL - Extra Packages for Enterprise Linux); or they may contain a mirror of the main repo.
Accessing these and installing software requires using your Linux distro's command line package management tool. The major ones you will encounter when you use Linux are:

- [APT (Aptitude Package Manager)](https://manpages.ubuntu.com/manpages/xenial/man8/apt.8.html): This is used in Debian based Linux distributions such as Ubuntu, Linux Mint, and Kali Linux.
- [DNF (Dandified Yum)](https://man7.org/linux/man-pages/man5/dnf.conf.5.html): Which is an update of YUM (Yellowdog Updater, Modified). Both of these operate as a front end for [RPM (Red Hat Package Manager)](https://man7.org/linux/man-pages/man8/rpm.8.html). These are used in Red Hat based Linux distros.
- [Pacman (Pacman Package Manager)](https://archlinux.org/pacman/pacman.8.html): This is used in Arch Linux, as well as a few others.
- [Zypper Package Manager](https://www.unix.com/man-page/suse/8/zypper/"): This is used in OpenSUSE, which is popular for use on servers in Europe.

Since you are using Ubuntu, you will be using **APT** to install software and update the system.

## Updating www and installing Apache

Start your **www** instance in the AWS Learner Lab, and connect to it using SSH. Once you have logged in, issue the following command to update your system.

```bash
sudo apt update && sudo apt upgrade
```

Now that your system is up to date it's time to install Apache, which is the software that will be powering your web server. In Ubuntu, the Apache package is called **apache2**. Additionally, you can use **apt** with the **-y** option to auto assume yes. This can save a little time when you know you want to install the software or updates.

```bash
sudo apt -y install apache2
```

## Managing Linux services

Normally the next thing you would want to do is start the service using the **systemctl** command, then confirm it is running. However, when you installed Apache2 this was automatically done. It's still a good idea to confirm the service is running. The **systemctl** command always requires elevated privileges, and follows the same format:

- start: Starts the service.
- stop: Stops the service.
- restart: Restarts the service.
- status: Displays information about the current status of the service, such as whether is running or not.
- enable: Configures the service to start automatically on boot.
- disable: Prevents the service from starting automatically on boot.

This isn't a complete list, but it covers all of the things you will require for most system administration tasks. Using what you have just learned, check to see if the apache2 service is running.

```bash
sudo systemctl status apache2
```

You should see the following output:
![Apache Running](/img/apache-running.png)

## Accessing your Apache server through web browser

It's time to test to see if everything's working properly. Browse to your instance details in EC2 by clicking on Instances, then the Instance ID next to your www instance (screenshot below).

![AWS Instance ID](/img/aws-instance-id.png)

Click open address beside your Public IPv4 address. You should see the default Apache test page (screenshot below). If you do not, edit your url and change **https://** to **http://**. You will learn how to configure **https** in later in this lab. Accessing your server through **http** will be fine until then.

![Apache Test Page](/img/apache-default.png)

## Implementing an elastic IP in AWS

When you are configuring network resources such as (routers, network printers or servers) you want them to have a static IP, which doesn't change. Currently our AWS instance pulls a new IP from Amazon's DHCP server every time it boots up. Fortunately, you can configure a static IP through what AWS calls an **Elastic IP**. These cost money when they're not in use, and will be the biggest expense item from our free $50 credits this semester (since your instance will be offline unless you are working on things for this course). To obtain an Elastic IP in EC2, click on **Elastic IPs** under **Network & Security**:
![Accessing the Elastic IP settings](/img/elastic-ip.png)

Then click **Allocate Elastic IP address** in the top right corner. On the bottom of the new screen, leave the rest of the defaults and click **Allocate** (screenshot below).
![Allocating an elastic IP](/img/elastic-ip2.png)

Now you've reserved your Elastic IP. It is yours for as long as you want (which will be the entire semester). However, you need to associate it with the instance you want to access it through (in this case, www). To do that check the box beside your Elastic IP, then click the **Actions** drop down and click **Associate Elastic IP address**.

Next, click on the **Instance** box and select the instance with **(www)** in the name (screenshot below), and click **Associate** in the bottom right corner.
![Associating an Elastic IP](/img/associate-elastic-ip.png)

Now you can access your www instance from anywhere by using the same IP address. You should write this IP down somewhere for future use. You will be mapping a domain name to it in the next section.

## Accessing your Apache server from a web browser

Open a web browser and either copy/paste, or type out your elastic IP in the address bar (make sure the url is being requested using **http** and not **https**) to confirm you can still access your Apache test page.

## Registering a domain name

In the web browser, go to [My.Custom.Domain](https://mycustomdomain.senecapolytechnic.ca/) and log in with your Seneca credentials. You will be using this to create an A record and map it to the elastic IP of your instance from Lab 3. If you do not have access please **contact your professor** so you can proceed.

### Creating an A record

Once you have logged in to [My.Custom.Domain](https://mycustomdomain.senecapolytechnic.ca/). You should see a screen similar to the one below.

![My.Custom.Domain login](/img/my-custom-domain-login.png)

Click **Create DNS Records**.

On the following screen, click **Create your first DNS Record!**, and fill in the following information (see the following screenshot for an example)

- **Name:** www
- **Type:** A Record (IPv4 Address)
- **Value:** _your elastic IP_
- **Course:** OPS345
- **Description:** Address record for www instance.

Click **Create**.

![Creating an address record](/img/dns-a-record.png)

### Testing Your DNS configuration

1. Launch the AWS Learner Lab and login. Make sure your www instance is running.
1. Next, login to your first instance and issue the following commands. Note the output of each. **Substitute your username in the provided commands**.

```bash
nslookup www.yourusername.mystudentproject.ca
```

and

```bash
dig www.yourusername.mystudentproject.ca
```

3. Access dig via the [Google Admin ToolBox](https://toolbox.googleapps.com/apps/dig/#A/) and enter the value **www.yourusername.mystudentproject.ca** into the Name field (make sure the record type is set to **A**). You should see output similar to the following:

![Google Admin ToolBox](/img/google-admin-toolbox.png)

4. Provided all of the above displayed the correct output, open a web browser and type **www.yourusername.mystudentproject.ca** (replace your username) in the URL bar of a web browser. This could be on your PC, or any device. You should see your website from Lab 3! If you don't, double check and make sure you see **http://** and not **https://**.

Make sure you see the correct output from the previous commands indicating your DNS is working before proceeding to the next step.

> Note: You can now login via SSH (from the command line) using your FQDN! 
>
> Use the command **ssh ubuntu@www.username.mystudentproject.ca**

## Preparing your system to generate and install an SSL certificate

Login to your **www** instance. You are going to install Certbot, which will automate configuring HTTPS using Let's Encrypt.

### Installing Certbot

First, check to see if it is available by issuing the following command.

```bash
sudo apt search certbot
```

You should see the following output.

![Confirming certbot is available to install with apt](/img/apt-search-certbot.png)

Once you have confirmed it is available, install it.

```bash
sudo apt -y install certbot python3-certbot-apache
```

### Configuring an Apache Virtual Host

Create the and edit a file for your virtual host configuration. You can use either vi or nano. Replace wwwusernamemystudentprojectca with your domain name, with the www and top level domain, but without the **dots(.)**. This will allow Certbot to find the correct VirtualHost block and update it.

```bash
sudo nano /etc/apache2/sites-available/wwwusernamemystudentprojectca.conf
```

Enter the following text (again, replacing the username with yours).

```bash
ServerName www.jasoncarman.mystudentproject.ca
```

Save your file and exit (**ctrl + x**).

### Testing and Reloading the Apache configuration

Enter the following command to test your Apache configuration.

```bash
sudo apache2ctl configtest
```

You should see a message indicating Syntax OK. If you don't, double check your file name and contents for errors. Sample output follows.

![apachectl configtest indicating syntax is ok](/img/apache2-configtest.png)

Now you can reload apache2 using systemctl.

```bash
sudo systemctl reload apache2
```

## Generating an SSL certificate using Let's Encrypt and Certbot

Now you are ready to generate your SSL certificate using Certbot. You are going to configure Apache to reconfigure and reload the configuration whenever necessary. This way you do not need to worry about updating your SSL certificate every 90 days, which is when certificates issued through Let's Encrypt and Certbot expire. Issue the following command:

```bash
sudo certbot --apache
```

At the email address prompt, enter your Seneca Polytechnic issued email.
![Running certbot](/img/sudocertbotapache.png)

Accept the terms of service. Answer as you wish for sharing your email, then enter your domain name. See the following example.
![Generating your certificate](/img/certbotregister.png)

Update your **Wordpress Website SG** security group rules to allow incoming HTTPS traffic from the anywhere IP: 0.0.0.0/0

## Testing your configuration

Open a web browser try to access your Apache test page using HTTPS. It should work! Your screen should look similar to the following.
![Website showing FQDN and HTTPS](/img/apachehttps.png)


## Lab 6 Sign-Off

Take screenshots showing the following:

- Your Elastic IP
- Accessing your Apache2 Ubuntu Default Page through a web browser using **https** and showing your fully qualified domain name (FQDN).

## Exploration Questions

1. What package manager does Ubuntu use?
1. How do you find the current status of the apache2 service?
1. What is an Elastic IP?
1. What is the difference between HTTP and HTTPS?
1. What port did you have to allow inbound in the **Wordpress Website SG** security group?
1. What service (command) did you use to generate your TLS/SSL certificate?
1. What is certbot?
