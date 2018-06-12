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
- [Select a server host](#select-a-server-host)
- [Set up server](#set-up-server)
- [Set up user](#set-up-user)
- [Set up SSH](#set-up-ssh)
- [Secure server](#secure-server)
- [Deploy app](#deploy-app)
  - [Clone app files](#clone-app-files)
  - [Configure Apache server](#configure-apache-server)
  - [Create Python virtual environment](#create-python-virtual-environment)
  - [Configure Flask app](#configure-flask-app)

## Select a server host

I started out configuring a server with [Amazon Lightsail](https://aws.amazon.com/lightsail/), as recommended by Udacity. I was not happy with the experience. In particular, when changing the SSH port, I got locked out and had to destroy and recreate the instance. I also am required to set up a user `grader` for this project, but was not able to log in with any user other than `ubuntu`.

<details><summary>Here are my notes on Amazon Lightsail.</summary>

## Set up server <!-- omit in toc -->

### Get server from Amazon Lightsail <!-- omit in toc -->

I set up AWS Lightsail Ubuntu as described in the [Udacity project docs](server-udacity-docs.md). The xip.io link provided in the docs doesn't work.

The Lightsail SSH client opens a terminal window in the browser. It frequently disconnects, and doesn't have command memory when restarting. I switched to my desktop SSH client (iTerm2 or Hyper) at this point.

### Connect with desktop SSH client <!-- omit in toc -->

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

### Update and upgrade Ubuntu packages <!-- omit in toc -->

```shell
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

### Set time zone to UTC  <!-- omit in toc -->

```shell
sudo dpkg-reconfigure tzdata
```

Select "None of these," then find UTC in the list and press enter.

## Set up user  <!-- omit in toc -->

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

## Set up SSH access for user  <!-- omit in toc -->

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

  - Ensure the ~/.ssh/config file is set up to allow public key authentication. See [askubuntu](https://askubuntu.com/questions/311558/ssh-permission-denied-publickey) for info.
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

## Secure server  <!-- omit in toc -->

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

## Prep server for deployment  <!-- omit in toc -->

## Deploy Flask app  <!-- omit in toc -->

</details>

The [DigitalOcean Initial Server Setup with Ubuntu 16.04 tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04) was far more helpful than any of the Amazon or Udacity resources. I switched to DigitalOcean for the rest of the project.

## Set up server

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
- If you ever need to restart the server, run `sudo shutdown -h now` from the console, then turn the server back on through the website interface.

## Set up user

- I continued by following the [DigitalOcean Initial Server Setup with Ubuntu 16.04 tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).
- The first step is logging in as root:

  ```shell
  ssh root@104.131.20.200
  ```

- At this point, root can log in with the password sent to your email address by DigitalOcean, and you can then change the password after login.
- After logging in as root, I created two users: `br3ndonland` for me and `grader` for the Udacity grader. I left the `grader` password `grader`. I gave each user `sudo` privileges.

  ```shell
  adduser br3ndonland
  adduser grader
  usermod -aG sudo br3ndonland
  usermod -aG sudo grader
  ```

## Set up SSH

- See the [DigitalOcean How To Connect To Your Droplet with SSH](https://www.digitalocean.com/community/tutorials/how-to-connect-to-your-droplet-with-ssh) guide.
- I [generated an SSH key and added it to the SSH agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) *on my local machine,* with the method recommended by GitHub, which attaches your email instead of the local machine name. The config file may need to be manually created with `touch ~/.ssh/config` first. I named the key `udacity_grader`.

  ```shell
  touch ~/.ssh/config
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  eval "$(ssh-agent -s)"
  ssh-add -K ~/.ssh/udacity_grader
  ```

- I copied the SSH key to the server for each user with `ssh-copy-id`. **Note that `ssh-copy-id` relies on password authentication, so if you disable password authentication this won't work.** Copy the SSH ID and verify login before disabling password authentication.

  ```shell
  ssh-copy-id -i ~/.ssh/udacity_grader br3ndonland@104.131.20.200 -p 2200
  ssh -p 2200 br3ndonland@104.131.20.200
  exit
  ssh-copy-id -i ~/.ssh/udacity_grader grader@104.131.20.200 -p 2200
  ssh -p 2200 grader@104.131.20.200
  exit
  ```

  - At this point, login will be accomplished by matching the *private* key that pairs with the public key you uploaded. The the ~/.ssh/config file on your local machine can also be configured.

    ```text
    Host udacity6
      Hostname 104.131.20.200
      User grader
      Port 2200
      PubKeyAuthentication yes
      IdentityFile ~/.ssh/udacity_grader

    Host *
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/id_rsa
    ```

  - If the config file is set up as above, log in with `ssh udacity6`
  - On my first try with DigitalOcean, I added an SSH key when initially creating the server instance. This made my life more difficult because I couldn't use `ssh-copy-id`. I destroyed the droplet, re-did droplet creation, and added the SSH key from the command line with `ssh-copy-id` after logging in.

## Secure server

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
  - Open the configuration file and edit the file with the nano text editor.

    ```shell
    sudo nano /etc/ssh/sshd_config
    ```

    ```text
    # What ports, IPs and protocols we listen for
    Port 2200
    ```

  - **Important**: Disable password authentication in `sshd_config` so SSH is required for login, and disable root login so user must specify `sudo`:

    ```text
    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication no
    PermitRootLogin no
    ```

  - Save and quit with ctrl+x.
  - Restart SSH

    ```shell
    sudo systemctl reload sshd
    service ssh restart
    ```

  - Exit and log back in, this time specifying port 2200.

    ```shell
    ssh grader@104.131.20.200 -p 2200
    ```

  - **The DigitalOcean browser console is still available after changing the ssh port, unlike Amazon Lightsail.**
    - Log into the DigitalOcean website.
    - Click the droplet (server).
    - Click console.
    - The login is slightly different. Instead of typing `ssh grader@104.131.20.200 -p 2200`, simply type `grader`.
    - This totally saved me. I started having SSH `publickey` issues, and was unable to ssh in from my local machine, but I could still log in through the browser console, so I didn't have to destroy the droplet and start over.
  - **After disabling password login, I started getting the SSH error `Permission denied (publickey).`** I spent hours troubleshooting it and working in the DigitalOcean browser console.  I read resources including [SSH.com](https://www.ssh.com/iam/ssh-key-management/) and [askubuntu](https://askubuntu.com/questions/311558/ssh-permission-denied-publickey), but the solutions didn't help. Eventually, I realized that the best solution was:
    - Delete the SSH key
    - Re-do the keygen as described above.
    - Temporarily enable password login again from the DigitalOcean browser console.
    - Add the new key with `ssh-copy-id` as described above, and authenticate with user password.
    - Log in with SSH.
    - Disable password authentication.
    - Exit
    - Log back in to verify SSH.

## Deploy app

DigitalOcean has helpful documentation for the Linux server itself, but less documentation for applications. They have [one-click app support for Django](https://www.digitalocean.com/community/tutorials/how-to-use-the-django-one-click-install-image-for-ubuntu-16-04), but not for Flask. There were a few community articles on Flask. I followed the [instructions](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04) to serve the Flask application with Gunicorn and Nginx on Ubuntu 16.04.

In the future, it would be preferable to just use [Docker](https://www.docker.com), or at least automate the server creation and app deployment process with shell scripts.

### Clone app files

- Git was already installed.

  ```shell
  $ which git
  /usr/bin/git
  ```

- Clone [app repo](https://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog) and create directory in a single step:

  ```shell
  sudo git clone git://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog.git /var/www/catalog
  ```

  The app is now located at

  ```text
  /var/www/catalog
  ```

- Change owner of catalog directory:

  ```shell
  sudo chown -R grader:grader catalog
  ```

- Add the client_secrets.json file, containing Google Cloud credentials.
  - Create an empty JSON file, and open the file in the nano text editor.

    ```shell
    sudo touch client_secrets.json
    sudo nano client_secrets.json
    ```

  - Copy the client secret from the JSON on your local machine (or your Google Cloud dashboard) into the file on the server.
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

### Configure Apache server

- Install and configure Apache to serve a Python `mod_wsgi` application.

  ```shell
  sudo apt-get install apache2
  sudo apt-get install libapache2-mod-wsgi-py3
  sudo a2enmod wsgi
  ```

- Navigating to [http://104.131.20.200/](http://104.131.20.200/) should show the "It works!" default Apache page.
- Add Apache [configuration file](https://httpd.apache.org/docs/2.2/configuring.html) `catalog.conf`

  ```shell
  sudo nano /etc/apache2/sites-available/catalog.conf
  ```

  ```text
  <VirtualHost *:80>
    ServerName 104.131.20.200
    ServerAdmin brendon.w.smith@gmail.com
    WSGIDaemonProcess catalog user=grader threads=3
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
      Order allow,deny
      Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- Enable virtual host (should only need to be done once):

  ```shell
  sudo a2ensite catalog
  ```

- If needed, logs can be viewed:

  ```shell
  sudo locate access.log access_log
  sudo nano /var/log/apache2/access.log
  ```

### Create Python virtual environment

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
- Initialize pipenv, specifying the path to python 3.6. Pipenv should install the package dependencies.

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

- The Flask app should now be running.
- Stop the server with ctrl+c and continue with configuration process.

### Configure Flask app

#### WSGI

- Configure WSGI file `catalog.wsgi`

  ```shell
  sudo nano /var/www/catalog/catalog.wsgi
  ```

  ```text
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/home/grader/var/www/catalog/")

  from catalog import app as application
  ```

- Create WSGI entry point: Basically create a new file to import Flask instance from application file (*application.py* in this case), and replicate the instructions at the bottom of the *application.py* file. Note that I had to create a new *wsgi.py* file first with `touch`, then add the text, otherwise it wouldn't save (I was working in the DigitalOcean browser console, not sure if that's why). I'm also not sure if you need the `CLIENT_SECRET` in the WSGI file, but I was getting `app.secret_key` errors from Gunicorn later so I added it.

  ```shell
  (pipenv) <PATH> $ touch /var/www/catalog/wsgi.py
  (pipenv) <PATH> $ sudo nano /var/www/catalog/wsgi.py
  ```

  ```python
  from application import app
  import json

  CLIENT_SECRET = json.loads(open('client_secrets.json', 'r')
                             .read())['web']['client_secret']

  if __name__ == 'main':
      app.secret_key = CLIENT_SECRET
      app.run(host='0.0.0.0', port=8000, debug=True)
  ```

#### Gunicorn and Nginx

- Install Gunicorn and Nginx

  ```shell
  sudo apt-get install nginx
  cd /var/www/catalog
  pipenv install gunicorn
  ```

- Create Gunicorn systemd unit file:

  ```shell
  sudo nano /etc/systemd/system/catalog.service
  ```

  ```text
  [Unit]
  Description=Gunicorn instance to serve catalog
  After=network.target

  [Service]
  User=grader
  Group=www-data
  WorkingDirectory=/var/www/catalog
  Environment="PATH=/home/grader/.local/share/virtualenvs/catalog-1BsMKvn0"
  ExecStart=/home/grader/.local/share/virtualenvs/catalog-1BsMKvn0/bin/gunicorn --workers 3 --bind unix:catalog.sock -m 007 wsgi:app
  ```

- Test Gunicorn:

  ```shell
  (pipenv) <PATH> $ gunicorn --bind 0.0.0.0:8000 wsgi:app
  ```

  Gunicorn appeared to start up, and was listening on the correct port, but I was not able to access the server at [http://104.131.20.200:80/](http://104.131.20.200:80/) at this point.

- Create nginx file for app:

  ```shell
  sudo nano /etc/nginx/sites-available/catalog
  ```

  ```text
  server {
      listen 80;
      server_name server_domain_or_IP;

      location / {
          include proxy_params;
          proxy_pass http://unix:/var/www/catalog.sock;
      }
  }
  ```

- The nginx configuration file shouldn't need to be changed.

  ```shell
  sudo nano /etc/nginx/nginx.conf
  ```

- Test with

  ```shell
  sudo nginx -t
  ```

  ```text
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etx/nginx/nginx.conf test is successful
  ```

- At this point, testing as recommended by the DigitalOcean guide is successful, both with and without the pipenv active. However attempting to complete the restart step listed in the DigitalOcean guide results in an error.

  ```shell
  sudo systemctl restart nginx
  ```

  ```text
  Job for nginx.service failed because the control process exited with error code. See "systemct1 status nginx.service" and "journalct1 -xe" for details.
  ```

- I couldn't understand the error log.
- When visiting the server public IP, I get

  ```text
  502 Bad Gateway
  nginx/1.10.3 (Ubuntu)
  ```

For notes on project review, see [server-review.md](server-review.md).

[(Back to top)](#top)
