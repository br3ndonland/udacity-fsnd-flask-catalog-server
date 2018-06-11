# Linux server deployment

<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="300" alt="Udacity logo">
</a>

Udacity Full Stack Web Developer Nanodegree program

[Project 6. Linux server deployment](https://github.com/br3ndonland/udacity-fsnd-p6-server)

Brendon Smith

[br3ndonland](https://github.com/br3ndonland)

## Table of Contents <!-- omit in toc -->

- [Project overview](#project-overview)
- [Project details](#project-details)
- [Project submission](#project-submission)

## Project overview

> You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.
>
> ### Why this project <!-- omit in toc -->
>
> A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.
>
> ### What will I learn <!-- omit in toc -->
>
> You will learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. You will then learn how to install and configure a web and database server and actually host a web application.
>
> ### How does this help my career <!-- omit in toc -->
>
> - Deploying your web applications to a publicly accessible server is the first step in getting users
> - Properly securing your application ensures your application remains stable and that your user’s data is safe

## Project details

> ### How will I complete this project <!-- omit in toc -->
>
> This project is linked to the [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299) course, which teaches you to secure and set up a Linux server. By the end of this project, you will have one of your web applications running live on a secure web server.
>
> To complete this project, you'll need a Linux server instance. We recommend using [Amazon Lightsail](https://lightsail.aws.amazon.com) for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.
>
> Webcasts for the Linux Server Configuration Project:
>
> Need some additional help getting started with the Linux Server Configuration Project, or simply curious and want to learn a little bit more? Watch the following Webcasts!
>
> These webcasts are recordings of live Q&A sessions and demos. As always, you should read the appropriate rubric for your project thoroughly before you begin work on any project and double check the rubric before submitting. The videos were made by Udacity's coaches. Think of them as extra supplemental materials.
>
> The webcasts for the Linux Server Configuration Project include:
>
> - [SSH: How to access a remote server and edit files](https://www.youtube.com/watch?v=HcwK8IWc-a8)
> - [Intro to TMux](https://www.youtube.com/watch?v=hZ0cUWWixqU)
> - [Deploying a Flask App with Heroku](https://www.youtube.com/watch?v=5UNAy4GzQ5E)
>
> ### Get your server <!-- omit in toc -->
>
> 1. Start a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com).
> 2. Follow the instructions provided to SSH into your server.
>
> #### Get started on Lightsail
>
> We're recommending [Amazon Lightsail](https://lightsail.aws.amazon.com) for this project. If you prefer, you can use any other service that gives you a publicly accessible Ubuntu Linux server. But Lightsail works pretty well and it's what we've tested.
>
> There are a few things you need to do when you create your server instance.
>
> 1. Log in
>     - First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
>
>       ![Amazon Web Services login page](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e45f9_screen-shot-2017-02-10-at-14.59.35/screen-shot-2017-02-10-at-14.59.35.png)
>       *Amazon Web Services login page.*
> 2. Create an instance
>     - Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
>
>       ![Lightsail screenshot: No instances](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e45a0_screen-shot-2017-02-10-at-14.58.17/screen-shot-2017-02-10-at-14.58.17.png)
>
>       *When you have no instances, Lightsail gives you a picture of an orange robot and suggests that you create an instance.*
> 3. Choose an instance image: Ubuntu
>     - Lightsail supports a lot of different instance types. An *instance image* is a particular software setup, including an operating system and optionally built-in applications.
>     - For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
>
>       ![Lightsail setup: Pick your instance image (Ubuntu)](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43c6_screen-shot-2017-02-10-at-14.46.15/screen-shot-2017-02-10-at-14.46.15.png)
>
>       *When you create an instance, Lightsail asks what kind you want.*
>       *For this project, choose an "OS Only" instance with Ubuntu.*
>
> 4. Choose your instance plan
>     - The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero.
>
>       ![Lightsail setup: Pricing options. (Choose $5/month with first month free.)](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43ca_screen-shot-2017-02-10-at-14.46.35/screen-shot-2017-02-10-at-14.46.35.png)
>
>       *Lightsail's options for instance pricing.*
>
>     - *For this project, pick the lowest one to get free-tier access.*
>     - **Be aware:** If you enable additional features in Lightsail, you may be charged extra for them.
> 5. Give your instance a hostname
>     - Every instance needs a unique hostname. You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Your instance's name will be visible to you and to the project reviewer.
>
>       ![Lightsail setup: Setting a name for your instance.](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43ce_screen-shot-2017-02-10-at-14.47.08/screen-shot-2017-02-10-at-14.47.08.png)
>
>       *I've named my instance `silly-name-here`.*
>
> 6. Wait for it to start up
>     - It may take a few minutes for your instance to start up.
>
>       ![Lightsail: Instance starting.](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43d2_screen-shot-2017-02-10-at-14.47.34/screen-shot-2017-02-10-at-14.47.34.png)
>
>       *While your instance is starting up, Lightsail shows you a grayed-out display.*
>
> 7. It's running, let's use it
>     - Once your instance is running, the display gets brighter.
>
>       ![Lightsail: Instance running.](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43cb_screen-shot-2017-02-10-at-14.48.29/screen-shot-2017-02-10-at-14.48.29.png)
>
>     - Once your instance has started up, you can log into it with SSH from your browser.
>
>       ![Lightsail: Main page for an instance.](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43c3_screen-shot-2017-02-10-at-14.48.40/screen-shot-2017-02-10-at-14.48.40.png)
>
>       *The main page for my `silly-name-here` instance. The big orange "Connect using SSH" button is the next step.*
>
>     - The public IP address of the instance is displayed along with its name. In the above picture it's `54.84.49.254`.
>     - **Note:** When you set up OAuth for your application, you will need a DNS name that refers to your instance's IP address. You can use the [xip.io](http://xip.io/) service to get one; this is a public service offered for free by Basecamp. For instance, the DNS name `54.84.49.254.xip.io` refers to the server above.
>     - Explore the other tabs of this user interface to find the Lightsail firewall and other settings. You'll need to configure the Lightsail firewall as one step of the project.
>     - When you SSH in, you'll be logged as the `ubuntu` user. When you want to execute commands as `root`, you'll need to use the `sudo` command to do it. [Review `sudo` here if you need a refresher!](https://youtu.be/fC6eWTMslt4)
>
>       ![Lightsail: SSH window.](https://d17h27t6h515a5.cloudfront.net/topher/2017/February/589e43be_screen-shot-2017-02-10-at-14.49.14/screen-shot-2017-02-10-at-14.49.14.png)
>
>       *An SSH window logged into the server instance. From here, it's just like any other Linux server.*
>
> 8. Project time
>     - Now that you have a working instance, you can get right into the project!
>
> ### Secure your server <!-- omit in toc -->
>
> 1. Update all currently installed packages.
> 2. Change the SSH port from **22** to **2200**. Make sure to configure the Lightsail firewall to allow it.
> 3. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
>
> *Warning:* When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. [Review this video](https://youtu.be/n3PC04x09gc) for details! When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.
>
> ### Give `grader` access <!-- omit in toc -->
>
> In order for your project to be reviewed, the grader needs to be able to log in to your server.
>
> 1. Create a new user account named `grader`.
> 2. Give `grader` the permission to `sudo`.
> 3. Create an SSH key pair for `grader` using the `ssh-keygen` tool.
>
> ### Prepare to deploy your project <!-- omit in toc -->
>
> 1. Configure the local timezone to UTC.
> 2. Install and configure Apache to serve a Python mod_wsgi application.
>     - If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: `sudo apt-get install libapache2-mod-wsgi-py3`.
> 3. Install and configure PostgreSQL:
>     - Do not allow remote connections
>     - Create a new database user named `catalog` that has limited permissions to your catalog application database.
> 4. Install `git`.
>
> ### Deploy the item catalog project <!-- omit in toc -->
>
> 1. Clone and setup your **Item Catalog** project from the Github repository you created earlier in this Nanodegree program.
> 2. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your `.git` directory is not publicly accessible via a browser!
>
> Happy Learning!

## Project submission

> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.
>
> *Note:* If you have no experience working in the shell we recommend starting with [Linux Command Line Basics](https://www.udacity.com/course/viewer#!/c-ud595-nd). Otherwise, you can jump straight into [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299).
>
> ### Submission <!-- omit in toc -->
>
> Please follow these steps to properly submit this project:
>
> 1. Create a new GitHub repository and add a file named `README.md`.
> 2. Your README.md file should include **all** of the following:
>     - The IP address and SSH port so your server can be accessed by the reviewer.
>     - The complete URL to your hosted web application.
>     - A summary of software you installed and configuration changes made.
>     - A list of any third-party resources you made use of to complete this project.
> 3. Locate the SSH key you created for the `grader` user.
> 4. During the submission process, paste the contents of the `grader` user's SSH key into the "Notes to Reviewer" field.
>
> When you're ready to submit your project, click [here](https://review.udacity.com/#!/projects/3573679011) and follow the instructions. Due to the high volume of submissions we receive, please allow up up to **7 business days** for your evaluation to be returned.
>
> If you are having any problems submitting your project or wish to check up on the status of your evaluation, please email us at **fullstack-project@udacity.com**.
>
> ### Evaluation <!-- omit in toc -->
>
> Your project will be evaluated by a Udacity Code Reviewer according to the rubric below. Be sure to review it thoroughly before you submit. All criteria must "meet specifications" in order to pass.
>
> [Click here to view the rubric](https://review.udacity.com/#!/projects/3573679011/rubric)
>
> ### Next Steps <!-- omit in toc -->
>
> You will get an email as soon as your reviewer has feedback for you. Congratulations on making it this far in the Nanodegree! You're almost finished!
