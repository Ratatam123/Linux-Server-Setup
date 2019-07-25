# Project 3: Linux Server Setup for HappyRent (Project 2)

## Purpose of this project

This Project is the final part of Udacity's *full stack webdevelopment* course.
It consists in the configuration of a Linux Server hosting the web app created
in the previous project of the course. 

## Prerequisites
* Virtual Box 
* Vagrant
* Python 3 installation in Vagrant environment
* (Unix style) Shell

## Aufgabenstellung README
A README file is included in the GitHub repo containing the following information: 
IP address, URL, summary of software installed, summary of configurations made, 
and a list of third-party resources used to complete this project.

### Basic Info
* Server name: Ubuntu_16.04
* Public IP: 18.196.69.164
* SSH Port: 2200

* Connection to server via terminal (after step 6 of setup):
```console
    ssh -i ~/.ssh/udacity_grader_key.rsa grader@18.196.69.164 -p2200
``` 

## Info: "During the submission process, 
paste the contents of the grader user's SSH key into the "Notes to Reviewer" field."





### Info

* The guide below presupposes that one has created an account on *Amazon Web Services* 
and a *lightsail* instance. Personally I chose Ubuntu 16.04 as 

* Info: Some of the steps are performed from a local Terminal window, others from
a Terminal window connected to the Ubuntu server via *ssh*. To distinguish the
two types, former will be tagged with *local*, letter with *remote* in the steps
described below.

* venv auch erwaehnen??

### Steps to take

* 1) Access the server from local Terminal
    
    1. Download the default ssh key from AWS => Account => SSH keys
    
    2. move the keys, named *LightsailDefaultKey-x.pem* with *x* being the location of 
    the server you chose (for me: *eu-central-1*)
    into local *~/.ssh* directory and rename them to *udacity_linux_key.rsa*
    
    ```console 
    mv ~/Downloads/LightsailDefaultKey-eu-central-1.pem ~/.ssh/udacity_linux_key.rsa
    ```
    3. Set file rights so that only the owner can write and read
    ```console
    chmod 600 ~/.ssh/udacity_linux_key.rsa
    ```
    4. Connect via ssh
    ```console
    ssh -i ~/.ssh/udacity_linux_key.rsa ubuntu@18.196.69.164
    ```

* 2) Open Port 2200 & restrict login to *ssh*

    1. Open the [image of the lightsail instance](https://lightsail.aws.amazon.com/ls/webapp/home/instances) in your Browser 
        * Click on the three vertical dots and choose the *Manage* option
        * Click the *Networking* tab and click on *Add another* in the *Firewall* section 
        * Choose following options: *Custom* *TCP* *2200*
    
    Source: [Stackoverflow](https://stackoverflow.com/questions/47342988/aws-ssh-port-timeout-after-changing-port-number)

    2. In the Terminal window connected to the instance open & update the config file (remote)
    
    ```console
    sudo nano /etc/ssh/sshd_config
    ```
    * Set *PasswordAuthentication* to *no*
    
    * Restart with updated config (remote)
    ```console
    sudo service ssh restart
    ``` 

    * Try connecting in another Terminal window from the local machine
    ```console
    ssh -i ~/.ssh/udacity_linux_key.rsa ubuntu@18.196.69.164 -p2200
    ```



* 3) Update packages (remote)
    
    ```console
    sudo apt-get update
    ```
    ```console
    sudo apt-get upgrade
    ```
    * Ubuntu 16.04:
    * pop-up "keep the local version currently installed"

    * install Finger (explain what for...?)

    ```console
    sudo apt-get install finger
    ```
    <!-- * Ubuntu 18.04:
    * Pop Up: " Restart services during package upgrades without asking? "
        * I chose "Yes" option

    * Pop Up: "A new version (/tmp/filein3TUB) of configuration file /etc/ssh/sshd_config is   │ 
    available, but the version installed currently has been locally modified."
        * I chose "keep the local version currently installed" -->

* 4) create user grader and grant him sudo permissions (remote)
    
    1. * creation of user
    ```console
    sudo adduser grader
    ```
        * Choose a password & type it in twice

    2. Define reading & writing rights:
        * Create a new file under the suoders directory
    
    ```console
    sudo nano /etc/sudoers.d/grader
    ```
        * add following line to the file & save it

    ```console
    grader ALL=(ALL:ALL) ALL
    ```
    "any user can run any command on any host as any user"
    * ohne 4

    <!-- In order to prevent the "sudo: unable to resolve host" error, edit the hosts:

    ```console
    sudo nano /etc/hosts.
    ```
    Add the host: 127.0.1.1 ip-10-20-37-65 -->

* 5) Set UTC and local time
    
    ```console
    sudo dpkg-reconfigure tzdata
    ```
    * I chose *Europe* & *Berlin*

    <!-- * EXTRA bei tomato (3.2) -->

* 6) Configure SSH-key authentication for grader (local & remote)
    
    1. Create ssh-key (local)
    
    ```console
    ssh-keygen -f ~/.ssh/udacity_grader_key.rsa
    ```

    2. * create directory *.ssh* for keys (remote)
    ```console
    sudo mkdir /home/grader/.ssh
    ```
    3. * open & copy contents from local udacity_grader_key.rsa.pub file (local)
    ```console
    cat ~/.ssh/udacity_grader_key.rsa.pub
    ```

    4. * create new file and paste them into it (remote)
    ```console
    sudo nano /home/grader/.ssh/authorized_keys
    ```
    * As a shortcut for 3. & 4., you can instead use the command [ssh-copy-id](https://www.ssh.com/ssh/copy-id)

    5. * define permissions (remote) (wiederholen was hier passiert!)

    ```console
    sudo chmod 700 /home/grader/.ssh
    sudo chmod 644 /home/grader/.ssh/authorized_keys
    ```
    [Info on permissions](http://www.thinkplexx.com/learn/article/unix/command/chmod-permissions-flags-explained-600-0600-700-777-100-etc)
    
    6. * and change ownership and group to grader (... root user to grader)
    ```console
    sudo chown -R grader:grader /home/grader/.ssh
    ```

    7. * logging in to lightsail instance is now enabled via: 
    ```console
    ssh -i ~/.ssh/udacity_grader_key.rsa grader@18.196.69.164 -p2200
    ```
    * all remaining steps should be performed via *grader*

* 7) Set up the *uncomplicated Firewall (ufw)* as user *grader*
Project requirements need the server to only allow incoming connections for SSH 
(port 2200), HTTP (port 80), and NTP (port 123).

    ```console
    sudo ufw default allow outgoing (vgl. Aufgabenstellung)
    sudo ufw allow ssh
    sudo ufw allow www
    sudo ufw allow 2200/tcp
    sudo ufw allow 123/udp
    sudo ufw allow 80/tcp
    ``` 

* 8) Installing Git
    1.
    ```console
    sudo apt-get install git
    ```

    2. Configuring username:
    ```console
    git config --global user.name <username>
    ```

    3. Configuring email:
    ```console
    config --global user.email <email>
    ``` 

* 9) Cloning the app 
    
    1. create directory for the app & change its ownership and group to grader

    ```console
    sudo mkdir /home/grader/webapp
    ```

    ```console
    sudo chown -R grader:grader /home/grader/webapp
    ```
    2. Move into the directory & clone the app

    ```console
    cd /home/grader/webapp
    ```

    ```console
    git clone https://github.com/Ratatam123/HappyRent.git HappyRent
    ```
    * Merke: 
        * alles in virtual environment packen
        * installation von requirements, 

## ab hier: C. Schafer

* 10) Creating virtual environment on server
    
    1. Installing & upgrading pip
    
    ```console
    sudo apt install --upgrade python3-pip
    ```

    2. Install module *venv* for creating virtual environment 
    
    ```console
    sudo apt install python3-venv
    ```

    3. Create virtual environment in cloned repository named *venv* & activate it

    ```console
    python3 -m venv /home/grader/webapp/happyrent venv
    ```
    ```console
    source venv/bin/activate
    ```

    * all following steps are done with the virtual environment activated

    4. Installing pip in venv, then install requirements

    ```console
    sudo apt install python3-pip
    ```
    ```console
    pip3 install -r requirements.txt
    ```
* 11) Set up config file on server to store configuration data for the *flask* app
(personal setup)

1. 
```console
sudo touch /etc/config.json
```

2. 
```console
sudo nano /etc/config.json
```

3. Write config info in json format

```json
{
        "SECRET_KEY": "579...",
        "GOOGLE_OAUTH_CLIENT_ID": "...",
        "GOOGLE_OAUTH_CLIENT_SECRET": "..."
}
```
* in config file from project folder (*/home/grader/webapp/happyrent/property_project/config.py*)

```python
...
import json
with open('/etc/config.json') as config_file:
        config = json.load(config_file)
class Config:
    SECRET_KEY = config.get('SECRET_KEY')
    ...
```


* 12) Installing & setting up *nginx* & *gunicorn*. Nginx processes static code (css, images etc.), 
gunicorn the Python code.

1. install nginx (global installiert) & gunicorn

```console
sudo apt install nginx
```

```console
pip3 install --user gunicorn
```

2. Remove default config files & create new one

*  for nginx

```console
sudo rm /etc/nginx/sites-enabled/default
```

```console
sudo nano /etc/nginx/sites-enabled/happyrent
```

* put in following setup: 

* generic

```
server {
    listen 80;
    server_name YOUR_IP_OR_DOMAIN;

    location /static {
        alias /home/YOUR_USER/YOUR_PROJECT/APP_DIRECTORY_NAME/static;
    }

    location / {
        proxy_pass http://localhost:8000;
        include /etc/nginx/proxy_params;
        proxy_redirect off;
    }
}
```

* specific
```
server {
    listen 80;
    server_name 18.196.69.164.xip.io;

    location /static {
        alias /home/grader/webapp/happyrent/property_project/static;
    }

    location / {
        proxy_pass http://localhost:8000;
        include /etc/nginx/proxy_params;
        proxy_redirect off;
    }
}
```

3. Update firewall & restart nginx

```console
sudo ufw allow http/tcp
```

<!-- * Folgendes rausnehmen, wenn 5000 nicht auch weiter oben geoeffnet
```console
sudo ufw delete allow 5000
``` -->

```console
sudo ufw enable
```

```console
sudo systemctl restart nginx
```


* 13) Set up posgresql (to replace sqlite used so far) 

1. Install postgres

* Install postgres & packages needed for Python to work with psql 

```console
sudo apt-get install libpq-dev python-dev
```

```console
pip3 install --user  psycopg2
```

* erste 3 Kommandos (killtheyak)
```console
sudo apt-get install postgresql postgresql-contrib, python-dev, libpython-dev, libevent-dev
```

* NOTWENDIG??
```console
sudo -u postgres createuser -s $USER
```


2. Open postgres, create a user & a database 

```console
sudo su - postgres
```
* open posgresql
```console
psql
```
* create user ==> user psql intern ODER = linux user?
```psql
CREATE USER happy_renter WITH PASSWORD 'best_rents';
```

* grant the user CREATEDB capability

```psql
ALTER USER happy_renter CREATEDB;
```
* create database

```psql
CREATE DATABASE happyrent_db WITH OWNER happy_renter;
```

* connect to database
```psql
\c happyrent_db
```
    connected as user 'postgres'??

* revoke rights for 'public'
```psql
REVOKE ALL ON SCHEMA public FROM public;
```

```psql
GRANT ALL ON SCHEMA public TO happy_renter;
```

<!-- * NEU:

```psql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO happy_renter;
``` -->

```psql
\q
```

```console
 exit
```

* replace address of old sqlite database address by new psql-db address in config.py

```console
sudo nano config.py
```

```python

SQLALCHEMY_DATABASE_URI = 'postgresql://happy_renter:best_rentsd@localhost/happyrent_db'
```

* Clean bug in routes.py appearing when using *postgres* instead of *sqlite* 

* OLD code

```python
@app.route("/")
@app.route("/home")
def home():
    property_types = []
    for type in db.session.query(PropertyType).distinct(PropertyType.type)\
            .group_by(PropertyType.type):
        property_types.append(type)

    return render_template('home.html', categories=property_types)
```

* NEW code: added *PropertyType.id* in *.group_by()*

```python
@app.route("/")
@app.route("/home")
def home():
    property_types = []
    for type in db.session.query(PropertyType).distinct(PropertyType.type)\
            .group_by(PropertyType.type, PropertyType.id):
        property_types.append(type)

    return render_template('home.html', categories=property_types) 
```



* 14) Add [xip.io](xip.io) to the public ip address as a substitute for a proper 
domain name to get Google OAuth working

1. Open custom Nginx config and add *xip.io* to public ip address as the servername

```console
sudo nano /etc/nginx/sites-enabled/happyrent
```

```
...
server_name 18.196.69.164.xip.io;
...
```

2. In [Google's developers console](https://console.developers.google.com/apis)
authorize 18.196.69.164.xip.io

* click on *Credentials* on the left side of the screen
* click on *OAuth consent screen* & add *xip.io* under *Authorized domains*
* click *Save*

* click on *Credentials* tab
* click on the project name (*flask_project* in my case)
* under *Authorized redirect URIs* add following URIs:
    * http://18.196.69.164.xip.io/google_login/google/authorized
    * http://18.196.69.164.xip.io/oauthcallback  
* click *Save*


* 14) Run the web app with *gunicorn*

* make sure you are in the following directory: */home/grader/webapp/happyrent*

```console
gunicorn -w 1 run:app
```

* Visit http://18.196.69.164.xip.io to see the web app in action


## Logging in

```
ssh -i ~/.ssh/udacity_grader_key.rsa grader@18.196.69.164 -p2200
```

```
cd webapp/happyrent
```

```
source venv/bin/activate
```


## References

Flask Video Tutorials by Corey Schafer:

&nbsp;&nbsp;&nbsp;&nbsp;[Flask Youtube tutorial series](https://www.youtube.com/watch?v=goToXTC96Co)
&nbsp;&nbsp;&nbsp;&nbsp;[nginx config](https://github.com/CoreyMSchafer/code_snippets/blob/master/Python/Flask_Blog/snippets/nginx.conf)
&nbsp;&nbsp;&nbsp;&nbsp;[supervisor config](https://github.com/CoreyMSchafer/code_snippets/blob/master/Python/Flask_Blog/snippets/supervisor.conf)


DNS via xip.io
&nbsp;&nbsp;&nbsp;&nbsp;[xip.io](https://serversforhackers.com/c/xipio-and-wildcard-subdomains-for-development)

postgresql

&nbsp;&nbsp;&nbsp;&nbsp;[Tutorial by DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04) 
&nbsp;&nbsp;&nbsp;&nbsp;[Tutorial by Steven Loria](http://killtheyak.com/use-postgresql-with-django-flask/)
&nbsp;&nbsp;&nbsp;&nbsp;[postgresql](https://medium.com/@nitinraturi/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-14-04-77146b5565a0)

Project Setup guides:

&nbsp;&nbsp;&nbsp;&nbsp;[By github user *stueken*](https://github.com/stueken/FSND-P5_Linux-Server-Configuration)
&nbsp;&nbsp;&nbsp;&nbsp;[By github user *Iliketomatoes*](https://github.com/iliketomatoes/linux_server_configuration)


### Bonus

* 14)  Install & set up *supervisor* to serve app automatically/permanently

```console
sudo apt install supervisor
```

* update config for supervisor
```console
sudo nano /etc/supervisor/conf.d/happyrent.conf
```

```
[program:happyrent]
directory=/home/grader/webapp/happyrent
command=/home/grader/webapp/happyrent/venv/bin/gunicorn -w 1 run:app
user=grader
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stderr_logfile=/var/log/happyrent/happyrent.err.log
stdout_logfile=/var/log/happyrent/happyrent.out.log
```

* create log-files
```console
sudo mkdir -p /var/log/happyrent
```
```console
sudo touch /var/log/happyrent/happyrent.err.log
```
```console
sudo touch /var/log/happyrent/happyrent.out.log
```

```console
sudo supervisorctl reload
```


