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

**Thank you for your very helpful review! Here is a summary of changes I made:**

- **Private SSH key**
  - I included the private SSH key on resubmission, so the reviewer could properly access the project without password authentication.
- **Disable password authentication**

  ```shell
  sudo nano /etc/ssh/sshd_config
  ```

  ```text
  # Change to no to disable tunnelled clear text passwords
  PasswordAuthentication no
  ```

  ```shell
  sudo service ssh restart
  ```

- **Web server responds on port 80**

  ```shell
  sudo locate access.log access_log
  sudo nano /var/log/apache2/access.log
  ```