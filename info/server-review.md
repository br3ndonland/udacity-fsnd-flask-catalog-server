# Project review

<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="300" alt="Udacity logo">
</a>

Udacity Full Stack Web Developer Nanodegree program

[Project 6. Linux server configuration](https://github.com/br3ndonland/udacity-fsnd-p6-server)

Brendon Smith

[br3ndonland](https://github.com/br3ndonland)

## Table of Contents <!-- omit in toc -->

- [First submission](#first-submission)
  - [Review](#review)
  - [Response](#response)

## First submission

### Review

#### Summary

> I know how confusing and frustrating start working with Keys, server configurations and all things related to this project can be, but don't give up! :muscle:
>
> PS: I was able to validate most of the specs because your server is allowing connections without the SSH key and this is a major security issue, check the Key-based SSH authentication is enforced. spec for more info.
>
> If you feel like I can be more helpful to you and other students in any manner, please use the feedback session to drop me a hint about it.
>
> Regards,
>
> Daniel

#### Rubric

##### User management

- [ ] The SSH key submitted with the project can be used to log in as grader on the server.
  > - Hi there, I can not use the key you've sent to log in to your server, we need the private key over the public one. Please submit your private key, not the public one. They are generated together, and the public key is like a lock on a door; everyone has access to it. Your private key is just like a key for a door, and not everyone has access to it. In other words, it only makes sense for you to send me the private key. [Here is an article](http://blakesmith.me/2010/02/08/understanding-public-key-private-key-concepts.html) that explains a little bit more in-depth about their difference. When you resubmit, just make sure to provide us with any passphrase or passwords we need.
- [x] You cannot log in as root remotely.
  > - The root user login is blocked accordingly to `sshd_config` file. Well done!

    ```shell
    $ cat /etc/ssh/sshd_config | egrep "PermitRootLogin"
    PermitRootLogin no
    ```

- [x] The grader user can run commands using sudo to inspect files that are readable only by root.
  > - Grader user can run commands under sudo. Very nice setup.

##### Security

- [x] Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  > - UFW status shows that the port configurations are correctly set. This step is essential to keep your server more secure.

    ```shell
    sudo ufw status
    ```

- [ ] Key-based SSH authentication is enforced.
  > - This task is one of the most important regarding security on your server. I was able to join by guessing the password without providing any SSH key.
  > - Passwords are not used to control this kind of access because they can be brute-forced, guessed or leaked, so an SSH implementation is a must to keep your server security at an acceptable level.
  > - If you are having trouble to set this up, please feel free to reach your classroom mentor, ask in the forums or even dropping a line in the feedback area of this review.
  > - [This thread](https://askubuntu.com/questions/346857/how-do-i-force-ssh-to-only-allow-users-with-a-key-to-log-in) may assist you if needed.
- [x] All system packages have been updated to most recent versions.
  > - Neat! System packages are on bleeding edge! Keeping dependencies up to date is very important to maintain overall system security and performance.
- [x] SSH is hosted on non-default port.
  > - Great job changing the default SSH port of your server. The sshd_config contains a lot of useful configurations to make your server more secure (or not, heh).
  > - [Here is a cheatsheet](http://www.cheat-sheets.org/saved-copy/OpenSSH_quickref.pdf) that helped me sometimes in this regard.

##### Application functionality

- [ ] The web server responds on port 80.
  > - There is an error preventing your web application to run at port 80. Please check the logs to see what is happening.
  > - [This link](https://unix.stackexchange.com/questions/38978/where-are-apache-file-access-logs-stored) may help you figure out what is the root cause of your app not being able to serve a page through / route.

    ```text
    Internal Server Error

    The server encountered an internal error or misconfiguration and was unable to complete your request.

    Please contact the server administrator at brendon.w.smith@gmail.com to inform them of the time this error occurred, and the actions you performed just before this error.

    More information about this error may be available in the server error log.

    Apache/2.4.18 (Ubuntu) Server at 104.131.20.200 Port 80
    ```

- [x] Database server has been configured to serve data (PostgreSQL is recommended).
  > Your server is hosting a data instance ready to be consumed by your Web app. Data is one of the most important things in a web application and sure is a requirement to work out there. Excellent work here!
- [x] Web server has been configured to serve the Item Catalog application as a WSGI app.
  > A WSGI Server can be challenging at first but once you get it it is just a matter of configuration. Congrats setting this up but keep in mind that there is a bunch of tools to make your web application available that you will get the chance to know while working with other projects/languages. Keep curious!

  ```shell
  $ grep .wsgi -nr /etc/apache2/sites-available
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
  ```

##### Documentation

- [x] A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project.
  > - Your README file looks fantastic! Very well detailed and structured. Such a good usage of Markdown. Keep it up!!
  > - You're already rocking but [here is a free course](https://br.udacity.com/course/writing-readmes--ud777) of writing READMEs which you can also share with your colleagues if you find it useful - Free as a speech anyways! :sunglasses:

#### Code review

> - [README.md](README.md)
>   - Great job highlighting critical info on your project!
>   - Great job so far, Brendon! :thumbsup:
>   - Good job setting up your server to be accessible through an IP address but most of the times a DNS setup is a must since asking users to write IP addresses all the time is usually cumbersome. [Here is a resource](http://www.steves-internet-guide.com/dns-guide-beginners/) containing some basic initial info about DNS.
>   - Good job adding a high-level system dependency list.
>   - Great job documenting some references. It is essential to make your documentation complete!
> - [server-methods.md](info/server-methods.md)
>   - Great walkthrough! Very well detailed.
>   - Excellent Markdown usage to make your README more readable! It sure makes project documentation stand out.

### Response

**Thank you for your helpful review. Here is a summary of changes I made:**

- [x] **Disable password authentication**
  - I opened the config file and changed `PasswordAuthentication` to `no`.

    ```shell
    sudo nano /etc/ssh/sshd_config
    ```

    ```text
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication no
    ```

  - I then restarted ssh.

    ```shell
    sudo service ssh restart
    ```

- [x] **Include SSH key**
  - I included the private SSH key on resubmission, so the reviewer could properly access the project without password authentication.
  - **I was not able to ssh in from my local machine, and had to complete the project in the DigitalOcean browser console.** I spent hours troubleshooting my ssh keys, but couldn't get past `Permission denied (publickey).`
  - I followed the suggestions on [askubuntu](https://askubuntu.com/questions/311558/ssh-permission-denied-publickey) and [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-custom-connection-options-for-your-ssh-client).
  - I tried modifying the *~/.ssh/config* file to include:

    ```text
    Host udacity-fsnd-p6-server
      Hostname 104.131.20.200
      User grader
      Port 2200
      PubKeyAuthentication yes

    Host *
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
    ```

  - I tried to log in with

    ```shell
    ssh udacity-fsnd-p6-server
    ```

  - and also directly, like

    ```shell
    ```

  - I modified the permissions on my .ssh directory and keys

    ```shell
    chmod 700 ~/.ssh
    ```

  - I tried re-generating the key. I even tried a more secure key, like the one used for GitHub.

    ```shell
    ssh-keygen -t rsa -b 4096 -C "brendon.w.smith@gmail.com"
    eval "$(ssh-agent -s)"
    ssh-add -K ~/.ssh/udacity_fsnd_p6_server
    pbcopy < ~/.ssh/udacity_fsnd_p6_server.pub
    ```

  - I added the key directly through the DigitalOcean browser interface.
  - I tried adding the key again with

    ```shell
    ssh-copy-id udacity-fsnd-p6-server
    ```

  - Nothing worked. No matter what I did, I still got

    ```text
    grader@104.131.20.200: Permission denied (publickey).
    ```

- [ ] **Web server on port 80**
  - The app needed more configuration to properly run Flask. I followed the [instructions from DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04) to serve the Flask application with Gunicorn and Nginx on Ubuntu 16.04.
  - I added notes on further configuration to [server-methods.md](server-methods.md).
  - Unfortunately I still could not get the app to run.
- [ ] **DNS**
  - [DigitalOcean is not a DNS registrar](https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-dns) at this time. I will consider purchasing a domain name in the future.