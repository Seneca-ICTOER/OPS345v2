---
id: lab8
title: Lab 8 - Simple Storage Service (S3) and Elastic Beanstalk
sidebar_position: 8
description: Backing up files using Simple Storage Service (S3) Buckets and Hosting Wordpress using AWS Elastic Beanstalk
---
# Lab 8 - Simple Storage Service (S3) Glacier and Elastic Beanstalk

## Overview
This week's lab will cover the following:

- Modifying the wordpress source code to prepare it for use in Elastic Beanstalk
- Creating a public Simple Storage Service (S3) bucket
- Uploading files to a Simple Storage Service (S3) bucket
- Creating a new RDS for use with Elastic Beanstalk
- Configuring Elastic Beanstalk
- Installing and configuring Wordpress

## Investigation 1: Wordpress Source Code Modification

Before we get into Elastic Beanstalk, we are going to set up a basic Wordpress configuration and store it in a zip file. Then we will use the AWS Simple Storage Service (s3) to create a bucket and store our Wordpress configuration in it. This will allow us to easily launch Wordpress inside the Elastic Beanstalk once the environment is set up. 

### Download and Unzip - Local Computer

1. On your local computer, download the current Wordpress source code from here: https://wordpress.org/latest.zip
1. Unzip the file. You should end up with a _wordpress_ directory. (Do not delete the original .zip file)

### Modify Wordpress Configuration File

#### Duplicate and Open Configuration File

1. In the local _wordpress_ folder, find a file called: **wp-config-sample.php**
1. Duplicate this file, and call it: **wp-config.php**
1. Open **wp-config.php** in a text editor. You will want something that supports syntax highlighting, such as Visual Studio Code.

#### Adding Database Connector Info as Environment Variables

In this file (wp-config.php), you will be adding database connector information as **_environment variables_**, not the actual connector information. (We'll add that information later.)

Find the following lines and add the bolded values:

1. define('DB_NAME', **getenv('DB_NAME')**);
1. define('DB_USER', **getenv('DB_USER')**);
1. define('DB_PASSWORD', **getenv('DB_PASSWORD')**);
1. define('DB_HOST', **getenv('DB_HOST')**);

#### Adding Authentication Unique Keys and Salts as Environment Variables

In the same file (wp-config.php), you'll be adding the authentication keys and salts as **_environment variables_**.

Find the following lines and add the bolded values:

1. define('AUTH_KEY', **getenv('AUTH_KEY')**);
1. define('SECURE_AUTH_KEY', **getenv('SECURE_AUTH_KEY')**);
1. define('LOGGED_IN_KEY', **getenv('LOGGED_IN_KEY')**);
1. define('NONCE_KEY', **getenv('NONCE_KEY')**);
1. define('AUTH_SALT', **getenv('AUTH_SALT')**);
1. define('SECURE_AUTH_SALT', **getenv('SECURE_AUTH_SALT')**);
1. define('LOGGED_IN_SALT', **getenv('LOGGED_IN_SALT')**);
1. define('NONCE_SALT', **getenv('NONCE_SALT')**);

![Image: Adding database connector information to wp-config.php.](/img/a2_wp-config-example.png)
_Figure 1: Adding database connector information to wp-config.php._

Confirm the values are correct and **save** the file.

### Zip As New File and Rename - Local Computer

> Warning: Make sure you **only** zip the wordpress directory, not the directory with the version number.

1. Find the **wordpress** folder on your local computer.
1. _Zip the entire wordpress directory_, not just the files inside. (Use the zip compression protocol. Don't use something else like .rar.)
1. Rename your new zip file: **wordpress-6.7.2-_modded_.zip** (Use whatever version the source zip file has.)

## Investigation 2: Creating a Simple Storage Service (S3) Bucket

Start your session in the Learner Lab by clicking on the Start Lab button. You are going to create an S3 bucket and upload your wordpress configuration files.

From the Console Home navigate to **Storage** > **S3**.

- Navigate to Amazon S3
- Click **Create bucket**

### General configuration
- General purpose: **selected**
- Bucket name: **senecausername-wordpress**

### Object Ownership
- ACLs disabled (recommended): **selected**

### Block Public Access settings for this bucket
- Block _all_ public access: **unchecked**
- I acknowledge that the current settings might result in this bucket and the objects within becoming public: **checked**

### Bucket versioning
- Enable

### Default encryption
- Accept the defaults

### Bucket key
- Enable

Scroll down and click **Create bucket**

Once your bucket has created, click on your **bucket's name** (ie: **candice-wordpress**).
- Click **Upload**
- Click **Add files**
- Select your wordpress-modded zip file

> Make note of the Destination location: ie: **s3://candice-wordpress**. You will need this for your next lab.

- Click **Upload**

Now that we have our Wordpress config ready to go in our S3 bucket, we can move on to creating our Elastic Beanstalk environment.

## Elastic Beanstalk Overview

When you have installed Wordpress previously, you simply uploaded the source code and the first time you load the webpage, provided the database connector information. However, Elastic Beanstalk applications are meant to be disposable.

Normally, when you add that database connector info, it is saved in a file called _wp-config.php_ on the webserver VM. This is fine for a traditional setup. However, **_in Elastic Beanstalk, changes made to static HTML or PHP are not saved if the Beanstalk application restarts_**, which it will do often. Whenever the application restarts, it will reload from the source zip file and the original, empty connector file. If you did this the traditional way, you'd have to constantly re-enter your DB connector info every time you started up your Learner Lab environment.

We _could_ add the DB connector info to _wp-config.php_ manually before we upload the source code, but there's a much better way.

We use **environment variables** to allow us to put all the info in the Elastic Beanstalk application wizard directly. That way, every time the application restarts and reloads from the source code zip, it'll then read our saved connector information from AWS itself. Read below for details and steps.

**Note:** All other information, like the Wordpress website name, users, theme settings, blog posts, etc., are saved in the actual database you created in RDS. This database does not get reset when the Elastic Beanstalk application restarts, so your actual blog data will remain intact.

## Investigation 3: Creating a RDS instance

Start your session in the Learner Lab by clicking on the **Start Lab** button. Once the red dot has turned green, click on it to enter the Learner Lab and access the AWS Console interface. You are going to create a new RDS instance.

From the **Console Home** navigate to **Database** > **Aurora and RDS**. See the following screenshot for reference.

![Relational Database](/img/rds.png)

Click **Create database** (part way down the screen). Use the following options.

1. Standard create
1. Engine type: **MariaDB**
1. Engine Version: **MariaDB 11.4.4**
1. Templates: **Sandbox**
1. DB instance identifier: **wordpress-elasticbeanstalk**
1. Master username: **admin**
1. Credentials management: **Self Managed**
1. Auto generate a password: **Checked**
1. DB instance class: **db.t3.micro**
1. Allocated storage: **5 GiB**
1. Enable storage autoscaling: **Unchecked**
1. Virtual private cloud (VPC): **Wordpress VPC**
1. DB subnet group: **Select the subnet group from Lab 5, it should be the auto selected one**
1. Public access: **No**
1. VPC security group: **Choose existing**
1. Existing VPC security groups:
   1. **Remove default VPC**
   1. **Add _Wordpress Database SG_** (look to see that it's there below the dropdown after you select it)
1. Availability Zone: **us-east-1a**
1. Monitoring > Enable Enhanced monitoring: **Accept the default, should be turned off**
1. Below the Monitoring section, Additional configuration > Initial database name: **wordpress** (Write the database name down! You will need this later.)
1. Enable automated backups: **Unchecked**
1. Enable encryption: **Unchecked**

Click **Create database**.

Close the popup window.

This will take a few minutes to create. Once the database has finished creating, click on the _View connection details_ button by the green success message at the top of the page. This gives you your database password.

Store the following connection information about your RDS instance in your lab logbook or a saved document. You'll need it later:

1. **Endpoint**
1. **Initial database name**
1. **Master username**
1. **Master password**

### Connecting to your database from www

Login to your **www** instance, and issue the following command to connect to your database. Be sure to substitute the credentials you wrote down earlier.

```bash
mysql -u admin -h **endpoint** -p
```

Enter your Master password when prompted. You should see the following screen indicating a successful connection.

![MariaDB connected](/img/mariadb-connect.png)

Issue the following command to display the databases.

```bash
show databases;
```

Disconnect from the database.

```bash
quit;
```

## Investigation 4: Setting up an Elastic Beanstalk environment

Navigate to **Compute** > **Elastic Beanstalk**. See the following screenshot for reference.

![Elastic Beanstalk](/img/elasticbeanstalk.png)

Click **Create application**, and use the following settings:

### Environment Tier

Select: **Web server environment**

### Application information

1. Application name: **wordpress**

### Environment information
1. Environment name: **Wordpress-env**

## Platform
1. Ensure **Managed platform** checked
1. Platform: **PHP**
1. Platform branch: **PHP 8.4** (or current latest)
1. Application code: **Upload your code**
1. Version label: **wordpress-6.7.2** (Use the version from your zip filename)
1. Public S3 URL: **https://username-wordpress.s3.us-east-1.amazonaws.com/wordpress-6.8.1-modded.zip** (Where **username** is **your Seneca username**)

## Presets
1. Presets: **Single instance (free tier eligible)**

Click next

### Configure Service Access

Select: Use an existing service role

1. Service role: **LabRole**
1. EC2 instance profile: **LabInstanceProfile**
1. EC2 key pair: **vockey**

Click next

### Set up networking, database and tags

1. VPC: **Wordpress VPC**

#### Instance Settings

1. Public IP address Activated: **Checked**
1. **Instance** subnets: **Public Subnet 1, Public Subnet 2** (both checked)


#### Database settings

1. **Database** subnets: **Private Subnet 1, Private Subnet 2** (both checked)

Click **Enable database**

1. Username: admin
1. Password: _The password you copied and wrote down earlier_

Click next

### Configure instance traffic and scaling

1. EC2 Security Groups: **Wordpress Website SG** & **Wordpress Database SG** (both checked)
1. Leave the rest default

Click next

### Configure updates, monitoring and logging

#### Monitoring

1. System: **Basic**

#### Managed platform updates

1. Managed updates: **Unchecked**

#### Email notifications

1. Email notification: **Add your Seneca email**

#### Platform Software

Before beginning this section, you will need two things:

1. Your database connector information (you saved this, right?)
1. Randomly generated auth keys and salts from here: https://api.wordpress.org/secret-key/1.1/salt/ (it's a good idea to save these in a text file, too)

#### Container Options

1. Proxy server: **Apache**
1. Document root: **/wordpress**
1. Click **Add environment property** and add the following **Environment properties**
   1. DB_HOST: **your RDS database URL**
   1. DB_NAME: **wordpress**
   1. DB_USER: **admin**
   1. DB_PASSWORD: **your auto-generated database password**
   1. AUTH_KEY: **(use gathered info from salt page)**
   1. SECURE_AUTH_KEY: **(use gathered info from salt page)**
   1. LOGGED_IN_KEY: **(use gathered info from salt page)**
   1. NONCE_KEY: **(use gathered info from salt page)**
   1. AUTH_SALT: **(use gathered info from salt page)**
   1. SECURE_AUTH_SALT: **(use gathered info from salt page)**
   1. LOGGED_IN_SALT: **(use gathered info from salt page)**
   1. NONCE_SALT: **(use gathered info from salt page)**

Hint: None of these values should have single quotes in them. (i.e. ')

![Image: Adding database connector information, auth keys and salts to your Elastic Beanstalk application as static Environment Variables.](/img/a2_beanstalk-environment-variables-example.png)
_Figure 2: Adding database connector information, auth keys and salts to your Elastic Beanstalk application as static **Environment Variables**._

Click next.

#### Review options

Review all settings and ensure they match the instructions above. Once you hit **Create**, the application will take several minutes to create.

### Create the application.

click **Submit** when ready.

While you wait for the creation to complete, check your e-mail to confirm your notification subscription.

If your application fails to build, double check your **wp-config.php** configuration file for syntax errors. Additionally, make sure when you rezipped the file you only select the **wordpress** folder, not the main folder.

## Investigation 5: Accessing Wordpress

Open the URL presented in the Wordpress EBS instance and begin the site setup.

### Site Information

Set the following site information:

1. Site Title: **Your name's blog**
1. Username: **_yourSenecaUsername_**
1. Password: **Choose a strong password** (do not reuse the DB password!)
1. Your Email: **_yourSenecaEmailAddress_**
1. Search engine visibility: **Unchecked**

> If you get a message indicating a failure to connect, make sure you zipped the **wordpress** folder and it's contents only. You can rezip the file and click **Upload and deploy** if necessary.

### Blog Post:

Add a blog post detailing the following:

- What did you think of this lab?
- What was the most difficult part for you?
- What was the easiest part for you?

## Lab 8 Sign-Off (Show Instructor)

Take screenshots showing the following:

- Your S3 bucket with your wordpress zip file in it
- Your blog post (including the URL in the browser)

Shutting down your database:
- Naviage to **Aurora and RDS** > **Databases**.
- Select the radio button beside **wordpress-db**
- Click on **Actions** > **Stop temporarily**

This will shutdown your database for 7 days and pause billing. You may need to repeat this.

Do not delete the database yet. You will need it to complete Assignment 2.

## Exploration Questions

1. What is S3?
1. Describe the process of creating an S3 bucket as outlined in the lab instructions.
1. What is Elastic Beanstalk?
1. How is this lab similar to the wordpress install in **Lab 6/7**? How is it different?
