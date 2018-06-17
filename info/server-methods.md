# Computational narrative

<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="300" alt="Udacity logo svg">
</a>

Udacity Full Stack Web Developer Nanodegree program

[Project 6. Linux server deployment](https://github.com/br3ndonland/udacity-fsnd-p6-server)

Brendon Smith

[br3ndonland](https://github.com/br3ndonland)

[![license](https://img.shields.io/badge/license-MIT-blue.svg?longCache=true&style=for-the-badge)](https://choosealicense.com/)

## Table of Contents <!-- omit in toc -->

- [Select a server host](#select-a-server-host)
- [Set up server](#set-up-server)
- [Set up user](#set-up-user)
- [Set up SSH](#set-up-ssh)
- [Secure server](#secure-server)
- [Deploy app](#deploy-app)
  - [Configure PostgreSQL](#configure-postgresql)
  - [Clone app files](#clone-app-files)
  - [Set up Python environment](#set-up-python-environment)
  - [Configure web server](#configure-web-server)
  - [Hosting](#hosting)
  - [Troubleshooting](#troubleshooting)

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
sudo apt update
sudo apt upgrade
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

[(Back to top)](#top)

## Set up server

- It was easy to set up my DigitalOcean droplet. I just followed the on-screen instructions, but there is also a [tutorial](info@news.digitalocean.com) available if needed.
- I chose a $5 base-level droplet, enabled the $1 backups, and paid for it with PayPal.
- I did not set up the SSH key during droplet creation. See below for SSH setup.
- Ready to go! Wow, that was so much easier than Amazon Lightsail.
- Update and upgrade packages, then reboot

  ```shell
  sudo apt update
  sudo apt upgrade
  sudo reboot
  ```

- Set time zone to UTC:

  ```shell
  sudo dpkg-reconfigure tzdata
  ```

  - Select "None of these," then find UTC in the list and press enter.
- If you ever need to restart the server, run either `sudo reboot`, or `sudo shutdown -h now` and turn the server back on through the website interface.

[(Back to top)](#top)

## Set up user

- I continued by following the [DigitalOcean Initial Server Setup with Ubuntu 16.04 tutorial](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04).
- The first step is logging in as root:

  ```shell
  ssh root@192.241.141.20
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
- I [generated an SSH key and added it to the SSH agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) *on my local machine,* with the method recommended by GitHub, which attaches your email instead of the local machine name. The config file may need to be manually created with `touch ~/.ssh/config` first. I named the key `udacity6`.

  ```shell
  touch ~/.ssh/config
  ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
  eval "$(ssh-agent -s)"
  ssh-add -K ~/.ssh/udacity6
  ```

- I copied the SSH key to the server for each user with `ssh-copy-id`. **Note that `ssh-copy-id` relies on password authentication, so if you disable password authentication this won't work.** Copy the SSH ID and verify login before disabling password authentication. If you have already reconfigured your ssh port to 2200, add `-p 2200`. Also, be sure to reference the *private* key when using `ssh-copy-id`, and not the public key (in this example, *udacity6* instead of *udacity6.pub*).

  ```shell
  ssh-copy-id -i ~/.ssh/udacity6 br3ndonland@192.241.141.20
  ssh br3ndonland@192.241.141.20
  exit
  ssh-copy-id -i ~/.ssh/udacity6 grader@192.241.141.20
  ssh grader@192.241.141.20
  exit
  ```

- At this point, login will be accomplished by matching the *private* key that pairs with the public key you uploaded.
- On my first try with DigitalOcean, I added an SSH key when initially creating the server instance. This made my life more difficult because I couldn't use `ssh-copy-id`. I destroyed the droplet, re-did droplet creation, and added the SSH key from the command line with `ssh-copy-id` after logging in.

[(Back to top)](#top)

## Secure server

- I did this as user `grader` with `sudo`.

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
    ssh grader@192.241.141.20 -p 2200
    ```
- The *~/.ssh/config* file on your local machine can also be configured for easier login. This file is on my local machine, so I was able to to open it with vscode.

  ```shell
  $ code ~/.ssh/config
  ```

  ```text
  Host udacity6
    Hostname 192.241.141.20
    User grader
    Port 2200
    PubKeyAuthentication yes
    IdentityFile ~/.ssh/udacity6

  Host *
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_rsa
  ```

- If the config file is set up as above, log in with `ssh udacity6`.
- **The DigitalOcean browser console is still available after changing the ssh port, unlike Amazon Lightsail.**
  - Log into the DigitalOcean website.
  - Click the droplet (server).
  - Click console.
  - The login is slightly different. Instead of typing `ssh grader@192.241.141.20 -p 2200`, simply type `grader`.
  - This totally saved me. I started having SSH `publickey` issues, and was unable to ssh in from my local machine. I could still log in through the online browser console, where I re-enabled password authentication, enabling me to log in from my local machine and re-do the ssh keygen process. I didn't have to destroy the droplet and start over.
- **After disabling password login, I started getting the SSH error `Permission denied (publickey).`** I couldn't log in from my local terminal. I spent hours troubleshooting it and working in the DigitalOcean browser console.  I read resources including [SSH.com](https://www.ssh.com/iam/ssh-key-management/) and [askubuntu](https://askubuntu.com/questions/311558/ssh-permission-denied-publickey), but the solutions didn't help. Eventually, I realized that the best solution was:
  - Delete the SSH key
  - Re-do the keygen as described above.
  - Temporarily enable password login again from the DigitalOcean browser console.
  - Add the new key with `ssh-copy-id` as described above, and authenticate with user password.
  - Log in with SSH.
  - Disable password authentication.
  - Exit
  - Log back in to verify SSH.

[(Back to top)](#top)

## Deploy app

DigitalOcean has helpful documentation for the Linux server itself, but less documentation for applications. They have [one-click app support for Django](https://www.digitalocean.com/community/tutorials/how-to-use-the-django-one-click-install-image-for-ubuntu-16-04), but not for Flask. There were a few community articles on Flask.

In the future, it would be preferable to just use [Docker](https://www.docker.com), or at least automate the server creation and app deployment process with shell scripts.

### Configure PostgreSQL

- My app uses SQLite, but apparently Postgres must be installed in order to create and manage the application database. Note the prompt changes after logging in to the database.

  ```shell
  $ sudo apt-get install libpq5
  $ sudo apt-get install postgresql
  $ sudo su - postgres
  postgres@udacity6:~$ psql
  psql (9.5.12)
  Type "help" for help.

  postgres=CREATE USER catalog WITH PASSWORD 'grader';
  CREATE ROLE
  postgres=ALTER USER catalog CREATEDB;
  ALTER ROLE
  postgres=CREATE DATABASE catalog WITH OWNER catalog;
  CREATE DATABASE
  postgres=\c catalog
  You are now connected to database "catalog" as user "postgres".
  catalog=REVOKE ALL ON SCHEMA public FROM public;
  REVOKE
  catalog=GRANT ALL ON SCHEMA public TO catalog;
  GRANT
  catalog=\q
  postgres@udacity6:~$ exit
  logout
  ```

- Changing from SQLite to PostgreSQL will require installation of `psycopg2` after Python is installed.

### Clone app files

- Git should already be installed.

  ```shell
  $ which git
  /usr/bin/git
  ```

- Clone [app repo](https://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog) and create directory in a single step:

  ```shell
  sudo git clone git://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog.git /var/www/catalog
  ```

  The app is now located in `/var/www/`, the directory in which Apache allows sites, or "virtual host configurations."

  ```text
  cd /var/www/catalog
  ```

- Change owner of catalog directory:

  ```shell
  sudo chown -R grader:grader /var/www/catalog
  ```

### Set up Python environment

#### Configure app files

- Copy the client secret from your [Google Cloud APIs credentials dashboard](https://console.cloud.google.com/apis/credentials) into the *client_secrets.json* file on the server. Make sure the server's IP, and your domain name if you have one, have been added to the "Authorized JavaScript origins" and "Authorized redirect URIs."

  ```shell
  cd /var/www/catalog
  sudo touch client_secrets.json
  sudo nano client_secrets.json
  ```

- Modify the *application.py* file for the server:
  - Move `app.secret_key` out of the `if __name__ == '__main__'` block (which will not be used by the WSGI server), as recommended [here](https://stackoverflow.com/a/33898263).
  - Change paths to the *client_secrets.json* file, and any other external files, to absolute. Remember to modify the path in the Google Sign-In code.

  ```shell
  cd /var/www/catalog
  sudo nano application.py
  ```

  ```python
  # Obtain credentials from JSON file
  CLIENT_ID = json.loads(open('/var/www/catalog/client_secrets.json', 'r')
                        .read())['web']['client_id']
  CLIENT_SECRET = json.loads(open('/var/www/catalog/client_secrets.json', 'r')
                            .read())['web']['client_secret']
  redirect_uris = json.loads(open('/var/www/catalog/client_secrets.json', 'r')
                            .read())['web']['redirect_uris']
  app.secret_key = CLIENT_SECRET


  # Google Sign-In
  @app.route('/gconnect', methods=['POST'])
  def gconnect():
      """App route function for Google Sign-In."""
      # Confirm that client and server tokens match
      if request.args.get('state') != login_session['state']:
          response = make_response(json.dumps('Invalid state parameter.'), 401)
          response.headers['Content-Type'] = 'application/json'
          return response
      # Obtain authorization code
      code = request.data
      try:
          # Upgrade the authorization code into a credentials object
          oauth_flow = flow_from_clientsecrets('/var/www/catalog/client_secrets.json', scope='')
          oauth_flow.redirect_uri = 'postmessage'
          credentials = oauth_flow.step2_exchange(code)
      except FlowExchangeError:
          response = make_response(
              json.dumps('Failed to upgrade the authorization code.'), 401)
          response.headers['Content-Type'] = 'application/json'
          return response
      # (application code continues below)
  ```

- Configure *database_setup.py*, *database_data.py*, and *application.py* for PostgreSQL instead of SQLite.

  ```python
  # delete this
  # engine = create_engine('sqlite:///catalog.db')
  # and add this
  engine = create_engine('postgresql://catalog:grader@localhost/catalog')
  ```

#### Set up Python

- Install Python 3.6
  - My virtual environment requires Python 3.6, which is not available from Ubuntu apt yet. Python 3.6 must be installed as a third-party application.
  - I followed the instructions [here](https://askubuntu.com/a/865569) to install:

    ```shell
    sudo add-apt-repository ppa:deadsnakes/ppa
    sudo apt update
    sudo apt install python3.6
    ```

  - Python 3.6 must be specified with `python3.6` when python is run, or when the virtual environment is configured.

- Install pip

  ```shell
  sudo apt install python3-pip
  ```

#### Run app without virtual environment

**Note that, despite extensive troubleshooting, I was not able to configure WSGI to serve up the Python Flask app from a virtual environment. The way I got the app running was by installing required packages without a virtual environment.**

- Install required packages

  ```shell
  sudo pip3 install flask
  sudo pip3 install oauth2client
  sudo pip3 install psycopg2
  sudo pip3 install requests
  sudo pip3 install sqlalchemy
  ```

- Run application files

  ```shell
  python3.6 database_setup.py
  python3.6 database_data.py
  python3.6 application.py
  ```

- The Flask app should now be running on localhost. Stop the local server with ctrl+c and continue with the configuration process.

<details><summary>Virtual environment configuration details</summary>

**I did not end up using the virtual environment, because I couldn't get WSGI to serve up my app from within a virtual environment. See Apache troubleshooting below.**

#### Virtual env

- Python 3 is bundled with the `venv` module for creation of virtual environments. I already had my GitHub repo set up with `venv`.
- The plan was to run this:

  ```shell
  cd <PATH>
  python3.6 -m venv venv
  # activate virtual env
  source venv/bin/activate
  # install modules listed in requirements.txt
  (venv) <PATH> pip install -r requirements.txt
  ```

- When attempting to run `python3.6 -m venv venv` I got an error:

  ```text
  Error: Command '['/var/www/catalog/v/bin/python3.6', '-Im', 'ensurepip', '--upgrade', '--default-pip']' returned non-zero exit status 1.
  ```

- I also had pipenv installed, so I tried that instead.

#### Pipenv

- Install pipenv

  ```shell
  sudo pip3 install pipenv
  ```

- Initialize pipenv, specifying the path to python 3.6. Pipenv will install the package dependencies.

  ```shell
  cd /var/www/catalog
  grader@udacity6:/var/www/catalog$ pipenv install --python /usr/bin/python3.6
  Creating a virtualenv for this project…
  Using /usr/bin/python3.6 (3.6.5) to create virtualenv…
  ⠋Running virtualenv with interpreter /usr/bin/python3.6
  Using base prefix '/usr'
  New python executable in /home/grader/.local/share/virtualenvs/catalog-1BsMKvn0/bin/python3.6
  Also creating executable in /home/grader/.local/share/virtualenvs/catalog-1BsMKvn0/bin/python
  Installing setuptools, pip, wheel...done.

  Virtualenv location: /home/grader/.local/share/virtualenvs/catalog-1BsMKvn0
  Installing dependencies from Pipfile.lock (9616db)…
    🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 24/24 — 00:00:24
  To activate this project's virtualenv, run the following:
  $ pipenv shell
  ```

- Spawn a pipenv shell

  ```shell
  pipenv shell
  ```

- Run application files. Note that, within the `pipenv`, it is not necessary to specify `python3.6`, because the env specifies the Python version.

  ```shell
  # ensure pipenv shell is activated
  (catalog-1BsMKvn0) grader@udacity6:/var/www/catalog$ python database_setup.py
  Database created.

  (catalog-1BsMKvn0) grader@udacity6:/var/www/catalog$ python database_data.py

  Provide credentials to be used when populating the database
  Please enter your name: Brendon Smith
  Please enter your email address: brendon.w.smith@gmail.com
  User brendon.w.smith@gmail.com successfully added to database.
  Category "Equipment" added to database.
  Category "Accessories" added to database.
  Item "Hoist Dual Action Leg Press" added to database.
  Item "Hoist Power Cage" added to database.
  Item "Pro Monster Mini Resistance Band" added to database.
  Item "Fat Gripz" added to database.
  Item "RumbleRoller" added to database.
  Item "SlingShot Hip Circle 2.0" added to database.
  Database population complete.

  (catalog-1BsMKvn0) grader@udacity6:/var/www/catalog$ python application.py
  * Serving Flask app "application" (lazy loading)
  * Environment: production
    WARNING: Do not use the development server in a production environment.
    Use a production WSGI server instead.
  * Debug mode: on
  * Running on http://0.0.0.0:8000/ (Press CTRL+C to quit)
  * Restarting with stat
  * Debugger is active!
  * Debugger PIN: 221-847-106
  ```

- The Flask app can also be run by pipenv without entering the pipenv shell:

  ```shell
  grader@udacity6:/var/www/catalog$ pipenv run application.py
  ```

- The Flask app should now be running on localhost. Stop the server with ctrl+c and continue with configuration process.

</details>

### Configure web server

#### WSGI

- Create WSGI script.
  - It seems like the script can either be named `catalog.wsgi` or `wsgi.py`, as long as the script name is properly specified in the Apache configuration file.
  - The script has a few parts:
    - Enabling the virtual environment: As explained above, I wasn't able to get WSGI to serve the app from the virtual environment, so I deleted this part. In the future, if configuration is successful, code added to *wsgi.py* would look something like this (based on the [Flask mod_wsgi docs](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/#working-with-virtual-environments), modified for pipenv):

      ```python
      # Enable use of virtual environment with mod_wsgi
      activate_this = '/home/grader/.local/share/virtualenvs/catalog-1BsMKvn0/bin/activate_this.py'
      with open(activate_this) as file_:
          exec(file_.read(), dict(__file__=activate_this))
      ```

    - Specifying the path to the application directory: It seems like this path is only used within *wsgi.py* to find the specific *application.py* file. File paths used within *application.py* will still need to be changed to absolute.
    - Importing the Flask app: The syntax requirements here were a little difficult to figure out also. I originally had `from catalog import app as application`, which didn't seem to be working. It helped to open the *wsgi.py* file in my local text editor (vscode), because the linting showed me that `catalog` was not found. I needed to import the Flask instance (`app`) from the application file *application.py*, which would be `from application import app as application`.
    - Name main block: This seems to be used instead of the name/main block in the main application file. I added the filesystem session type as recommended [here](https://stackoverflow.com/a/33898263).
- Here's what a completed WSGI script should look like:

  ```shell
  sudo nano /var/www/catalog/wsgi.py
  ```

  ```python
  # Add path to application directory
  import sys
  sys.path.insert(0, "/var/www/catalog")

  # Import Flask instance from main application file
  from application import app as application

  if __name__ == '__main__':
      app.config['SESSION_TYPE'] = 'filesystem'
      app.run(host='0.0.0.0', port=8000)

  ```

#### Apache

- Install and configure Apache to serve a Python `mod_wsgi` application. The [Flask docs](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/) have some simple instructions.
- Install Apache and Python 3 `mod_wsgi`

  ```shell
  sudo apt-get install apache2
  sudo apt-get install libapache2-mod-wsgi-py3
  ```

  - Some guides also suggest running `sudo a2enmod wsgi`, but WSGI should already enable itself upon installation.
- Browsing to the server ip [http://192.241.141.20/](http://192.241.141.20/) should show the "It works!" default Apache page.
  > Apache2 Ubuntu Default Page
  >
  > It works!
  >
  > This is the default welcome page used to test the correct operation of the Apache2 server after installation on Ubuntu systems.
- Add Apache [configuration file](http://httpd.apache.org/docs/current/configuring.html) `catalog.conf`.
  - `ServerName`: Server's IP address.
  - `ServerAdmin`: Optionally enter your email address here.
  - `WSGIDaemonProcess`: Helps the app run when logged in as a different user.
  - `WSGIScriptAlias`: There is an initial `/`, followed by the absolute file path to the WSGI script.
  - The `Directory` section grants access to the directory containing the application files.
  - The `ErrorLog` and `CustomLog` are particularly helpful for debugging.
- Here's how the completed Apache configuration file will look:

  ```shell
  sudo nano /etc/apache2/sites-available/catalog.conf
  ```

  ```text
  <VirtualHost *:80>
    ServerName 192.241.141.20
    ServerAdmin brendon.w.smith@gmail.com
    WSGIDaemonProcess application user=grader threads=3
    WSGIScriptAlias / /var/www/catalog/wsgi.py
    <Directory /var/www/catalog/>
      WSGIProcessGroup application
      WSGIApplicationGroup %{GLOBAL}
      Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- Enable virtual host (should only need to be done once):

  ```shell
  sudo a2ensite catalog
  sudo service apache2 restart
  ```

### Hosting

- DigitalOcean is not a DNS registrar at this time. I followed the [DigitalOcean DNS tutorial](https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-dns) to add a domain name purchased through [Hover](https://www.hover.com/).
- From my Hover dashboard, I pointed the domain to DigitalOcean's nameservers.

  ```text
  ns1.digitalocean.com
  ns2.digitalocean.com
  ns3.digitalocean.com
  ```

- In my DigitalOcean account, from the Networking tab, I set an A record so that Hostname catalog.br3ndonland.com directs to 192.241.141.20.
- DigitalOcean doesn't offer SSL either. I will look into SSL in the future.

### Troubleshooting

- If the site isn't working, check logs for errors:

  ```shell
  sudo tail /var/log/apache2/error.log
  sudo tail /var/log/apache2/access.log
  ```

[(Back to top)](#top)

<details><summary>Apache troubleshooting</summary>

#### Apache troubleshooting <!-- omit in toc -->

- I get an error when browsing to the server URL at [http://192.241.141.20/](http://192.241.141.20/):

  ```text
  Internal Server Error

  The server encountered an internal error or misconfiguration and was unable to complete your request.

  Please contact the server administrator at brendon.w.smith@gmail.com to inform them of the time this error occurred, and the actions you performed just before this error.

  More information about this error may be available in the server error log.
  Apache/2.4.18 (Ubuntu) Server at 192.241.141.20 Port 80
  ```

- I checked the error log. The most helpful things I could find were:

  ```shell
  sudo nano /var/log/apache2/error.log
  ```

  ```text
  File "/var/www/catalog/catalog.wsgi", line 7, in <module>
  from application import app as application
  ImportError: No module named 'application'
  mod_wsgi (pid=1781): Target WSGI script '/var/www/catalog/catalog.wsgi' cannot be loaded as Python module.
  mod_wsgi (pid=2078): Exception occurred processing WSGI script '/var/www/catalog/catalog.wsgi'.
  ```

- I tried changing permissions and it didn't change anything.

  ```shell
  sudo chmod 644 /etc/apache2/sites-available/catalog.conf
  sudo service apache2 restart
  ```

- At this point, I submitted the project again. See [server-review.md](server-review.md).
- The reviewer recommended the [DigitalOcean How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) guide. I had seen it before, but I went through it with the "Hello, World!"-style demo FlaskApp. I couldn't even get the demo app to run. The comments show five years of developers confused and frustrated by the tutorial. See below in "DigitalOcean demo FlaskApp."
- I spent some time going back through documentation, hoping to find an answer.
- I found a Stack Overflow thread "[How to install & configure mod_wsgi for py3](https://stackoverflow.com/questions/19344252/how-to-install-configure-mod-wsgi-for-py3)". I had already installed `libapache2-mod-wsgi-py3`. The top answer links out to the [Django 1.8 docs on mod_wsgi](https://docs.djangoproject.com/en/1.8/howto/deployment/wsgi/modwsgi/), which in turn laud the mod_wsgi docs:
  > The [official mod_wsgi documentation](http://modwsgi.readthedocs.org/) is fantastic; it’s your source for all the details about how to use mod_wsgi.
- Navigating to the [mod_wsgi project status page](http://modwsgi.readthedocs.io/en/develop/project-status.html) tells a different story:
  > In general, the documentation is in a bit of a mess right now and somewhat outdated...
- The [updated Django 2.0 docs](https://docs.djangoproject.com/en/2.1/howto/deployment/wsgi/modwsgi/) are less complementary:
  > The [official mod_wsgi documentation](https://modwsgi.readthedocs.io/) is your source for all the details about how to use mod_wsgi.
- Apparently the project is less fantastic than it used to be. None of this is giving me much confidence.
- The error message `Target WSGI script '/var/www/catalog/catalog.wsgi' cannot be loaded as Python module` suggests that maybe I need to reconfigure the WSGI script as a Python module. I changed the file to *wsgi.py* and updated the Apache configuration file accordingly. Still got the same error message.
- I started noticing a strange error indicating that the SQLAlchemy module was missing:

  ```shell
  grader@udacity6:/var/www/catalog$ sudo tail /var/log/apache2/error.log
  [Sat Jun 16 21:57:03.568120 2018] [wsgi:error] [pid 2503:tid 140707532621568] [client 151.76.182.223:49313]     from sqlalchemy import create_engine
  [Sat Jun 16 21:57:03.568151 2018] [wsgi:error] [pid 2503:tid 140707532621568] [client 151.76.182.223:49313] ImportError: No module named 'sqlalchemy'
  ```

- This was a clue that Apache wasn't making use of my virtual environment.
- **Although it is optimal to have a defined virtual environment for the app, after all the difficulty I have been through up to this point, I decided to just install the required modules without the virtual environment. I made sure *application.py* was running on localhost, restarted Apache, navigated to the IP address for the server, and... it worked!**

  ```shell
  sudo pip3 install flask
  sudo pip3 install oauth2client
  sudo pip3 install psycopg2
  sudo pip3 install requests
  sudo pip3 install sqlalchemy
  python3.6 application.py
  sudo service apache2 restart
  ```

- **FINALLY DONE!**

</details>

<details><summary>DigitalOcean demo FlaskApp</summary>

#### DigitalOcean demo FlaskApp <!-- omit in toc -->

- [DigitalOcean How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps) guide.
- Walked through the steps exactly.
- Built the demo app in a different directory on the same server (udacity6, 192.241.141.20).
- Here's how *FlaskApp.conf* looks:

  ```shell
  sudo nano /etc/apache2/sites-available/FlaskApp.conf
  ```

  ```text
  <VirtualHost *:80>
      ServerName http://192.241.141.20
      ServerAdmin admin@mywebsite.com
      WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
      <Directory /var/www/FlaskApp/FlaskApp/>
        Order allow,deny
        Allow from all
      </Directory>
      Alias /static /var/www/FlaskApp/FlaskApp/static
      <Directory /var/www/FlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- I modified the *FlaskApp.wsgi* file to attempt to activate the virtual env, because the tutorial's WSGI file doesn't properly direct to the venv:

  ```shell
  sudo nano /var/www/FlaskApp/flaskapp.wsgi
  ```

  ```python
  activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
  with open(activate_this) as file_:
      exec(file_.read(), dict(__file__=activate_this))

  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/")

  from FlaskApp import app as application
  application.secret_key = 'Add your secret key'
  ```

- Still nothing.
- I got to the step where I was reloading apache, and got

  ```shell
  grader@udacity6:/var/www/FlaskApp/FlaskApp$ service apache2 reload
  ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
  Authentication is required to reload 'apache2.service'.
  Multiple identities can be used for authentication:
  1.  Brendon Smith,,, (br3ndonland)
  2.  Udacity Grader,,, (grader)
  Choose identity to authenticate as (1-2): 2
  Password:
  ==== AUTHENTICATION COMPLETE ===
  Job for apache2.service failed because the control process exited with error code. See "systemctl status apache2.service" and "journalctl -xe" for details.
  grader@udacity6:/var/www/FlaskApp/FlaskApp$ systemctl status apache2.service
  ● apache2.service - LSB: Apache2 web server
    Loaded: loaded (/etc/init.d/apache2; bad; vendor preset: enabled)
    Drop-In: /lib/systemd/system/apache2.service.d
            └─apache2-systemd.conf
    Active: active (running) (Result: exit-code) since Thu 2018-06-14 01:33:12 UTC; 25min ago
      Docs: man:systemd-sysv-generator(8)
    Process: 2602 ExecStop=/etc/init.d/apache2 stop (code=exited, status=0/SUCCESS)
    Process: 2978 ExecReload=/etc/init.d/apache2 reload (code=exited, status=1/FAILURE)
    Process: 2628 ExecStart=/etc/init.d/apache2 start (code=exited, status=0/SUCCESS)
      Tasks: 61
    Memory: 15.1M
        CPU: 1.885s
    CGroup: /system.slice/apache2.service
            ├─2647 /usr/sbin/apache2 -k start
            ├─2649 /usr/sbin/apache2 -k start
            ├─2650 /usr/sbin/apache2 -k start
            └─2651 /usr/sbin/apache2 -k start
  grader@udacity6:/var/www/FlaskApp/FlaskApp$ sudo journalctl -xe
  --
  -- Unit apache2.service has begun reloading its configuration
  Jun 14 01:58:05 udacity6 apache2[2978]:  * Reloading Apache httpd web server apache2
  Jun 14 01:58:06 udacity6 apache2[2978]:  *
  Jun 14 01:58:06 udacity6 apache2[2978]:  * The apache2 configtest failed. Not doing anything.
  Jun 14 01:58:06 udacity6 apache2[2978]: Output of config test was:
  Jun 14 01:58:06 udacity6 apache2[2978]: AH00526: Syntax error on line 8 of /etc/apache2/sites-enabled/catalog.conf:
  Jun 14 01:58:06 udacity6 apache2[2978]: WSGIRestrictStdout not allowed here
  Jun 14 01:58:06 udacity6 apache2[2978]: Action 'configtest' failed.
  Jun 14 01:58:06 udacity6 apache2[2978]: The Apache error log may have more information.
  Jun 14 01:58:06 udacity6 systemd[1]: apache2.service: Control process exited, code=exited status=1
  Jun 14 01:58:06 udacity6 systemd[1]: Reload failed for LSB: Apache2 web server.
  -- Subject: Unit apache2.service has finished reloading its configuration
  -- Defined-By: systemd
  -- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
  --
  -- Unit apache2.service has finished reloading its configuration
  --
  -- The result is failed.
  Jun 14 01:58:06 udacity6 polkitd(authority=local)[1682]: Unregistered Authentication Agent for unix-process:2965:326266
  Jun 14 01:58:18 udacity6 sudo[2994]:   grader : TTY=pts/2 ; PWD=/var/www/FlaskApp/FlaskApp ; USER=root ; COMMAND=/bin/n
  Jun 14 01:58:18 udacity6 sudo[2994]: pam_unix(sudo:session): session opened for user root by grader(uid=0)
  Jun 14 01:58:23 udacity6 sudo[2994]: pam_unix(sudo:session): session closed for user root
  Jun 14 01:58:50 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:17:f0:08:00 SRC=185.208.209
  Jun 14 01:59:27 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:1b:f0:08:00 SRC=119.10.58.2
  Jun 14 01:59:45 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:1b:f0:08:00 SRC=134.119.179
  Jun 14 01:59:50 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:1b:f0:08:00 SRC=134.119.179
  Jun 14 02:00:06 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:17:f0:08:00 SRC=139.162.178
  Jun 14 02:01:02 udacity6 kernel: [UFW BLOCK] IN=eth0 OUT= MAC=aa:5d:6d:bc:91:c3:cc:e1:7f:a8:1b:f0:08:00 SRC=134.119.179
  Jun 14 02:01:35 udacity6 sudo[3007]:   grader : TTY=pts/2 ; PWD=/var/www/FlaskApp/FlaskApp ; USER=root ; COMMAND=/bin/j
  Jun 14 02:01:35 udacity6 sudo[3007]: pam_unix(sudo:session): session opened for user root by grader(uid=0)
  ```

- Finally an error I can work with. It looked like *catalog.conf* was actually throwing the error. At this time, in /etc/apache2/sites-enabled/catalog.conf I had

  ```text
  <VirtualHost *:80>
    ServerName http://192.241.141.20
    ServerAdmin brendon.w.smith@gmail.com
    WSGIDaemonProcess application user=grader threads=3
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/>
      WSGIScriptReloading On
      WSGIRestrictStdout Off
      WSGIProcessGroup application
      WSGIApplicationGroup %{GLOBAL}
      Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

- I deleted `WSGIRestrictStdout Off`. Reload successful.

  ```shell
  grader@udacity6:/var/www/FlaskApp/FlaskApp$ service apache2 reload
  ==== AUTHENTICATING FOR org.freedesktop.systemd1.manage-units ===
  Authentication is required to reload 'apache2.service'.
  Multiple identities can be used for authentication:
  1.  Brendon Smith,,, (br3ndonland)
  2.  Udacity Grader,,, (grader)
  Choose identity to authenticate as (1-2): 2
  Password:
  ==== AUTHENTICATION COMPLETE ===
  ```

- Can't see anything. Went through the comments on the tutorial. Edited the *hosts* file and added `192.241.141.20 flaskapp.dev`:

  ```shell
  $ sudo nano /etc/hosts
  # Your system has configured 'manage_etc_hosts' as True.
  # As a result, if you wish for changes to this file to persist
  # then you will need to either
  # a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
  # b.) change or remove the value of 'manage_etc_hosts' in
  #     /etc/cloud/cloud.cfg or cloud-config from user-data
  #
  127.0.1.1 udacity6 udacity6
  127.0.0.1 localhost
  192.241.141.20 flaskapp.dev
  # The following lines are desirable for IPv6 capable hosts
  ::1 ip6-localhost ip6-loopback
  fe00::0 ip6-localnet
  ff00::0 ip6-mcastprefix
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
  ff02::3 ip6-allhosts
  ```

- Still nothing. I get the same internal server error.
- Using `tail` was more helpful, but I couldn't clearly identify what was going wrong.

  ```shell
  grader@udacity6:~$ sudo service apache2 restart
  grader@udacity6:~$ sudo tail /var/log/apache2/error.log
  [Thu Jun 14 22:54:06.426781 2018] [mpm_event:notice] [pid 7836:tid 140718488364928] AH00489: Apache/2.4.18 (Ubuntu) mod_wsgi/4.3.0 Python/3.5.2 configured -- resuming normal operations
  [Thu Jun 14 22:54:06.426826 2018] [core:notice] [pid 7836:tid 140718488364928] AH00094: Command line: '/usr/sbin/apache2'
  [Thu Jun 14 22:54:13.187874 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818] mod_wsgi (pid=7839): Target WSGI script '/var/www/FlaskApp/flaskapp.wsgi' cannot be loaded as Python module.
  [Thu Jun 14 22:54:13.187971 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818] mod_wsgi (pid=7839): Exception occurred processing WSGI script '/var/www/FlaskApp/flaskapp.wsgi'.
  [Thu Jun 14 22:54:13.188308 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818] Traceback (most recent call last):
  [Thu Jun 14 22:54:13.188342 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818]   File "/var/www/FlaskApp/flaskapp.wsgi", line 10, in <module>
  [Thu Jun 14 22:54:13.188347 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818]     from FlaskApp import app as application
  [Thu Jun 14 22:54:13.188355 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818]   File "/var/www/FlaskApp/FlaskApp/__init__.py", line 1, in <module>
  [Thu Jun 14 22:54:13.188359 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818]     from flask import Flask
  [Thu Jun 14 22:54:13.188379 2018] [wsgi:error] [pid 7839:tid 140718390884096] [client 24.13.227.94:50818] ImportError: No module named 'flask'
  ```

- At this point, the demo app was just interfering with my actual project, so I turned it off.

  ```shell
  sudo a2dissite FlaskApp
  sudo service apache2 restart
  ```

- The tutorial definitely has holes, especially related to the virtual env. Here are a few comments:
  - > newaccount September 21, 2013: Correct me if I'm wrong but you create a virtualenv and then promptly set up an app which doesn't use the virtualenv... right?
  - > acronymcreations May 6, 2017: How can I check if the WSGI is actually loading the virtualenv? I'm pretty sure this is what is causing my problems.

</details>

<details><summary>Gunicorn and Nginx</summary>

#### Gunicorn and Nginx <!-- omit in toc -->

I followed the [instructions](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04) to serve the Flask application with Gunicorn and Nginx on Ubuntu 16.04. I didn't get it to work.

- Install Gunicorn and Nginx

  ```shell
  sudo apt-get install nginx
  cd /var/www/catalog
  pipenv install gunicorn
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

  Gunicorn appeared to start up, and was listening on the correct port, but I was not able to access the server at [http://192.241.141.20:80/](http://192.241.141.20:80/) at this point.

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

- Viewing the logs just shows a 500 server error and isn't helpful.
- When visiting the server public IP, I get

  ```text
  502 Bad Gateway
  nginx/1.10.3 (Ubuntu)
  ```

</details>

<details><summary>Azure</summary>

#### Azure <!-- omit in toc -->

- Review the page on [Configuring Python with Azure App Service Web Apps](https://docs.microsoft.com/en-us/azure/app-service/web-sites-python-configure).
- Select [Web app with Flask on Linux](https://portal.azure.com/#create/PTVS.FlaskLinux).
- I instructed Azure to pull the app from my GitHub repo. It seemed to work, but I wasn't sure how to configure the server from there.

</details>

For notes on project review, see [server-review.md](server-review.md).

[(Back to top)](#top)
