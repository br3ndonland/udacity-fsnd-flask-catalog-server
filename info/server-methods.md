# Computational narrative

<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="300" alt="Udacity logo svg">
</a>

Udacity Full Stack Web Developer Nanodegree program

[Project 6. Linux server deployment](https://github.com/br3ndonland/udacity-fsnd-p6-server)

Brendon Smith

[br3ndonland](https://github.com/br3ndonland)

[![license](https://img.shields.io/badge/license-MIT-blue.svg?longCache=true&style=for-the-badge)](https://choosealicense.com/)

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Steps](#steps)
  - [Select a server host](#select-a-server-host)
  - [Set up server](#set-up-server)
  - [Set up user](#set-up-user)
  - [Secure server](#secure-server)
  - [Deploy app](#deploy-app)

## Steps

### Select a server host

I started out configuring a server with [Amazon Lightsail](https://aws.amazon.com/lightsail/), as recommended by Udacity. I was not happy with the experience. In particular, when changing the SSH port, I got locked out and had to destroy and recreate the instance. I also am required to set up a user `grader` for this project, but was not able to log in with any user other than `ubuntu`.

<details><summary>Here are my notes on Amazon Lightsail.</summary>

### Set up server <!-- omit in toc -->

#### Get server from Amazon Lightsail <!-- omit in toc -->

I set up AWS Lightsail Ubuntu as described in the [Udacity project docs](server-udacity-docs.md). The xip.io link provided in the docs doesn't work.

The Lightsail SSH client opens a terminal window in the browser. It frequently disconnects, and doesn't have command memory when restarting. I switched to my desktop SSH client (iTerm2 or Hyper) at this point.

#### Connect with desktop SSH client <!-- omit in toc -->

```shell
ssh -i ~/.ssh/udacity_key.pem ubuntu@52.14.148.231
```

I saved the private key file to `~/.ssh`, and attempted to connect to the server. My connection attempt was rejected because the private key was unprotected. Note that the IP was for a previous instance that I got locked out of after I changed the port. I had to destroy it and and create a new one.

```shell
$ ssh -i ~/.ssh/udacity_key.pem ubuntu@52.14.148.231

@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
```

I knew the `chmod` command from prior experience, but wasn't sure which permissions were needed. Based on a Stack Exchange [superuser post](https://superuser.com/a/1261781), I changed permissions to 600, tried logging in again, and it worked.

```shell
cd ~/.ssh
chmod 600 udacity_key.pem
ssh -i ~/.ssh/udacity_key.pem ubuntu@52.14.148.231
```

Note that Lightsail will not allow login from root. Running `sudo` will permit root-level changes.

Logout with `exit` when done.

#### Update and upgrade Ubuntu packages <!-- omit in toc -->

```shell
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

#### Set time zone to UTC  <!-- omit in toc -->

```shell
sudo dpkg-reconfigure tzdata
```

Select "None of these," then find UTC in the list and press enter.

### Set up user  <!-- omit in toc -->

- Install `finger` to easily read user info:

  ```shell
  sudo apt install finger
  ```

- Create a new user account named `grader`.

  ```shell
  sudo adduser grader
  ```

  - In lesson 2, in the "Connecting as the New User" video, the instructor Mike Wales just makes his UNIX password the same as the name of the new user. I made the UNIX password "grader."
  - If you make a mistake, and need to delete the user, run `sudo deluser --remove-home grader`.
  - You can also enter `sudo ls -al` to see users and permissions.

- Give `grader` the permission to sudo.
  - Create a file with nano

    ```shell
    sudo nano /etc/sudoers.d/grader
    ```

  - In the file, add `grader ALL=(ALL:ALL) ALL`.
  - Save and quit with ctrl+x.
  - Run `sudo nano /etc/hosts` and confirm that the file contains `127.0.0.1 localhost`.

### Set up SSH access for user  <!-- omit in toc -->

- Create an SSH public/private key pair for `grader` using the ssh-keygen tool.
  - Open another terminal window, and don't log in to the server.
  - Generate ssh key on local machine with `ssh-keygen`.

    ```shell
    ssh-keygen -f ~/.ssh/udacity_key_grader
    ```

    - See [ssh.com](https://www.ssh.com/ssh/copy-id) for syntax.
    - Also see the [Digital Ocean tutorial on SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604).
    - As explained in lesson 2 part 23, "[Generating Key Pairs](https://youtu.be/afOCp8d60U8)," key pairs must be generated locally. Key pairs generated on a server may not be private.
    - Leave passphrase empty.

- Copy the public key (`udacity_key_grader.pub`) from local machine to remote Ubuntu server with `ssh-copy-id` and user `ubuntu`.

  ```shell
  ssh-copy-id -i ~/.ssh/udacity_key_grader.pub ubuntu@52.14.148.231
  ```

  - **You must ssh with the default `ubuntu` user. The new `grader` user does not yet have login permissions.** If you try to copy the key or log in as `grader` prior to installing the public key, you will probably get the following error:

    ```shell
    $ ssh-copy-id -i ~/.ssh/udacity_key_grader.pub grader@52.14.148.231
    grader@52.14.148.231: Permission denied (publickey).
    ```

  - The Udacity lessons and Ubuntu docs elsewhere on the internet don't clearly explain this restriction. I found [this serverfault thread](https://serverfault.com/questions/39733/why-do-i-get-permission-denied-publickey-when-trying-to-ssh-from-local-ubunt) that pointed me in the right direction.
  - At this point, you should be able to ssh with the default `ubuntu` user and the *private* key that pairs with the public key you uploaded. Attempting to log in as `grader` at this point will still result in `Permission denied (publickey).`

    ```shell
    $ ssh -i ~/.ssh/udacity_key_grader -p 22 grader@52.14.148.231
    grader@52.14.148.231: Permission denied (publickey).
    $ ssh -i ~/.ssh/udacity_key_grader -p 22 ubuntu@52.14.148.231
    Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-1060-aws x86_64)
    ```

  - After copying the key from local to server with `ssh-copy-id`, it should be located in `/root/.ssh/authorized_keys`.
  - Compare the local and remote public keys to verify.
    - Local:

      ```shell
      nano ~/.ssh/udacity_key_grader.pub
      ```

    - Remote:

      ```shell
      sudo nano /root/.ssh/authorized_keys
      ```

    - You should see the same text after `ssh-rsa`.
- Make a `.ssh` directory for user `grader` and copy the public key from user `ubuntu` to user `grader`.

  ```shell
  sudo mkdir /home/grader/.ssh
  sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
  ```

  - Verify that the public key has copied from user `ubuntu` to user `grader`.

    ```shell
    sudo nano /home/grader/.ssh/authorized_keys
    ```

### Secure server  <!-- omit in toc -->

Configure Uncomplicated Firewall (UFW) and set ports. Also see [Ubuntu Server Guide Firewall](https://help.ubuntu.com/lts/serverguide/firewall.html) page. **Leave this until after the user config. It's easy to get locked out when changing the port.**

- Check status: Currently inactive.
- Deny incoming connections by default.
- Allow outgoing connections by default.

  ```shell
  sudo ufw status
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  ```

- Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

  ```shell
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  ```

- Change SSH port from 22 to 2200
  - Open the configuration file

    ```shell
    ubuntu@ip-172-26-10-184:~$ sudo nano /etc/ssh/sshd_config
    ```

  - Edit the file with the nano text editor, save and quit with ctrl+x.
  - **Danger**: As the [Udacity project docs](server-udacity-docs.md) explain, deleting port 22 disables the Lightsail SSH client.

    > *Warning:* When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. [Review this video](https://youtu.be/n3PC04x09gc) for details! When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

    The video only shows how to configure ufw ports.

    "Those instructions" on the "Connect" tab of the server instance page don't provide much explanation:

    > Connect using your own SSH client
    >
    > You can connect to your instance using the following address and user name:
    >
    > Public IP
    > 52.14.148.231
    >
    > User name
    > ubuntu
    >
    > When you connect with your client, you will also need the private key.
    >
    > You configured this instance to use udacity_key (us-east-2) key pair.
    >
    > You can download your default private key from the Account page.
  - I got locked out once, and had to destroy the server instance. After this, plus the annoyance with SSH keys, I switched to DigitalOcean.
- Enable firewall

  ```shell
  sudo ufw enable
  ```

- Check firewall status

  ```text
  ubuntu@ip-172-26-10-184:~$ sudo ufw status numbered
  Status: active

      To                         Action      From
      --                         ------      ----
  [ 1] 2200/tcp                   ALLOW IN    Anywhere
  [ 2] 80/tcp                     ALLOW IN    Anywhere
  [ 3] 123/udp                    ALLOW IN    Anywhere
  [ 4] 2200/tcp (v6)              ALLOW IN    Anywhere (v6)
  [ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
  [ 6] 123/udp (v6)               ALLOW IN    Anywhere (v6)
  ```

### Prep server for deployment  <!-- omit in toc -->

### Deploy Flask app  <!-- omit in toc -->

</details>

The [DigitalOcean Initial Server Setup with Ubuntu 16.04 tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04) was far more helpful than any of the Udacity resources. I switched to DigitalOcean for the rest of the project.

### Set up server

- It was easy to set up my DigitalOcean droplet. I just followed the on-screen instructions, but there is also a [tutorial](info@news.digitalocean.com) available if needed.
- I chose a $5 base-level droplet, enabled the $1 backups, and paid for it with PayPal.
- I did not set up the SSH key during droplet creation. See below for SSH setup.
- Ready to go! Wow, that was so much easier than Amazon Lightsail.
- Update and upgrade packages

  ```shell
  sudo apt-get update
  sudo apt-get upgrade
  ```

- Set time zone to UTC:

  ```shell
  sudo dpkg-reconfigure tzdata
  ```

  - Select "None of these," then find UTC in the list and press enter.

### Set up user

- I continued by following the [DigitalOcean Initial Server Setup with Ubuntu 16.04 tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).
- After logging in as root, I created two users: `br3ndonland` for me and `grader` for the Udacity grader. I left the `grader` password `grader`. I gave each user `sudo` privileges.

  ```shell
  adduser br3ndonland
  adduser grader
  usermod -aG sudo br3ndonland
  usermod -aG sudo grader
  ```

- I set up SSH key access for each user with `ssh-copy-id`.

  ```shell
  ssh-keygen
  ssh-copy-id br3ndonland@104.131.20.200
  ssh br3ndonland@104.131.20.200
  exit
  ssh-copy-id grader@104.131.20.200
  ssh grader@104.131.20.200
  ```

  - On my first try, I added an SSH key when initially creating the server instance. This made my life more difficult for two reasons: I couldn't use `ssh-copy-id`, and I had a problem with the key. I had created a separate `udacity_key_grader`, but continued to get `Permission denied (publickey).` I re-did the keygen, just keeping the default `id_rsa` filename, and it worked after that.

### Secure server

- I did this as `root`, but I could have also done it as a user with `sudo`.

  ```shell
  sudo ufw app list
  sudo ufw allow OpenSSH
  sudo ufw allow 2200
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  sudo ufw enable
  ```

  ```text
  $ sudo ufw status
  Status: active

  To                         Action      From
  --                         ------      ----
  OpenSSH                    ALLOW       Anywhere
  2200/tcp                   ALLOW       Anywhere
  80/tcp                     ALLOW       Anywhere
  123/udp                    ALLOW       Anywhere
  2200                       ALLOW       Anywhere
  OpenSSH (v6)               ALLOW       Anywhere (v6)
  2200/tcp (v6)              ALLOW       Anywhere (v6)
  80/tcp (v6)                ALLOW       Anywhere (v6)
  123/udp (v6)               ALLOW       Anywhere (v6)
  2200 (v6)                  ALLOW       Anywhere (v6)
  ```

- Change SSH port from 22 to 2200
  - We were required to do this for the Udacity project.
  - Open the configuration file. Edit the file with the nano text editor, save and quit with ctrl+x.

    ```shell
    ubuntu@ip-172-26-10-184:~$ sudo nano /etc/ssh/sshd_config
    ```

  - **Important**: Disable password authentication in `sshd_config` so SSH is required for login:

    ```text
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication no
    ```

  - **Important**: Disable root login so user must specify `sudo`:

    ```text
    PermitRootLogin no
    ```

  - Restart SSH

    ```shell
    service ssh restart
    ```

  - Open a new terminal window and log in, this time specifying port 2200.

    ```shell
    ssh grader@104.131.20.200 -p 2200
    ```

### Deploy app

DigitalOcean has helpful documentation for the Linux server itself, but less documentation for applications. There were a few community articles on Flask, but they were several years old. Also, in the future, it would be preferable to automate the server creation and app deployment process with shell scripts. [Here](https://github.com/turkerdotpy/django-setup-digitalocean) is an example for DigitalOcean and Django.

- Install and configure Apache to serve a Python mod_wsgi application.

  ```shell
  sudo apt-get install apache2
  sudo apt-get install libapache2-mod-wsgi-py3
  sudo a2enmod wsgi
  ```

- Navigating to [http://104.131.20.200/](http://104.131.20.200/) should show the "It works!" default Apache page.
- Configure postgresql

  ```text
  sudo apt-get install libpq-dev python-dev
  sudo apt-get install postgresql postgresql-contrib
  sudo su - postgres
  psql
  CREATE USER catalog WITH PASSWORD 'password';
  ALTER USER catalog CREATEDB;
  CREATE DATABASE catalog WITH OWNER catalog;
  \c catalog
  REVOKE ALL ON SCHEMA public FROM public;
  GRANT ALL ON SCHEMA public TO catalog;
  \q
  exit
  ```

- Check contents of postgresql configuration file with

  ```shell
  nano /etc/postgresql/9.5/main/pg_hba.conf
  ```

- Git was already installed.

  ```shell
  $ which git
  /usr/bin/git
  ```

- Clone [app repo](https://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog) and create directory in a single step:

  ```shell
  sudo git clone git://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog.git /var/www/catalog
  ```

- Change owner of catalog directory:

  ```shell
  sudo chown -R grader:grader catalog
  ```

- Add the client_secrets.json file.
  - Create an empty JSON file, and open the file in the nano text editor.

    ```shell
    sudo touch client_secrets.json
    sudo nano client_secrets.json
    ```

  - Copy the client secret from the JSON on your local machine into the file on the server.
- Add WSGI file `catalog.wsgi`

  ```shell
  sudo nano /var/www/catalog/catalog.wsgi
  ```

  ```text
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  ```

- Add [configuration file](https://httpd.apache.org/docs/2.2/configuring.html) `catalog.conf`

  ```shell
  sudo nano /etc/apache2/sites-available/catalog.conf
  ```

  ```text
  <VirtualHost *:80>
    ServerName 104.131.20.200
    ServerAdmin brendon.w.smith@gmail.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
      Order allow,deny
      Allow from all
    </Directory>
    Alias /static /var/www/catalog/static
    <Directory /var/www/catalog/static/>
      Order allow,deny
      Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- Enable virtual host:

  ```shell
  sudo a2ensite catalog
  ```

- Install pip and pipenv

  ```shell
  sudo apt install python-pip
  sudo pip install pipenv
  ```

- Install Python 3.6
  - My pipenv requires Python 3.6, which is not available from Ubuntu apt yet. Python 3.6 must be installed as a third-party application.
  - I followed the instructions [here](https://askubuntu.com/a/865569) to install:

    ```shell
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt-get update
    sudo apt-get install python3.6
    ```

  - Python 3.6 must be specified with `python3.6` when python is run.
- Initialize pipenv, specifying the path to python 3.6.

  ```shell
  cd /var/www/catalog
  pipenv install --python /usr/bin/python3.6
  ```

- Spawn a pipenv shell

  ```shell
  pipenv shell
  ```

- Run application files

  ```shell
  # ensure pipenv shell is activated
  python3.6 database_setup.py
  python3.6 database_data.py
  python3.6 application.py
  ```