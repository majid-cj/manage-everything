# How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 18.04

# 1 initial server setup guide.

Step 1 — Logging in as Root
    <strong>$ ssh root@your_server_ip</strong>

Step 2 — Creating a New User
    $ adduser <new_user_name>

    #-------------------------------------------------------
    <strong>You will be asked a few questions, starting with the account password</strong>
    <strong>Enter a strong password and, optionally, fill in any of the additional
            information if you would like. This is not required and you can just
            hit ENTER in any field you wish to skip.</strong>

Step 3 — Granting Administrative Privileges
    $ usermod -aG sudo <new_user_name>

# dont foreget to do the following steps with the new user you created

    $ su <new_user_name>

# 2 Installing the Packages from the Ubuntu Repositories

<strong>using Django with Python 3, type:</strong>

    $ sudo apt update
    $ sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl

<strong>using Django with Python 2, type:</strong>
    $ sudo apt update
    $ sudo apt install python-pip python-dev libpq-dev postgresql postgresql-contrib nginx curl

<strong> Creating the PostgreSQL Database and User </strong>
    $ sudo -u postgres psql
    <strong>following commands in postgres termianl</strong>
    postgres=# CREATE USER <db_user_name> WITH PASSWORD 'password';
    postgres=# CREATE DATABASE <db_name>;
    postgres=# ALTER ROLE <db_user_name> SET client_encoding TO 'utf8';
    postgres=# ALTER ROLE <db_user_name> SET default_transaction_isolation TO 'read committed';
    postgres=# ALTER ROLE <db_user_name> SET timezone TO 'UTC';
    postgres=# GRANT ALL PRIVILEGES ON DATABASE <db_name> TO <db_user_name>;
    postgres=# \q;

# Creating a Python Virtual Environment for your Project

<strong> for python 3 <strong>
    $ sudo -H pip3 install --upgrade pip
    $ sudo -H pip3 install virtualenv


<strong> for python 2 <strong>
    $ sudo -H pip install --upgrade pip
    $ sudo -H pip install virtualenv

With virtualenv installed, we can start forming our project. Create and move
into a directory where we can keep our project files:

    $ mkdir ~/<project dir name>
    $ cd ~/<project dir name>

Within the project directory, create a Python virtual environment by typing:

    $ virtualenv <project virtual env name>
    # preferred to named it venv or env
    $ source <project_virtual_env_name>/bin/activate

after you activte your virtual environment, you move your project files here,
and if you hosted them at Github,Gitlab .. etc, you can initail a git repo here

    (<project_virtual_env_name>)$ git init

then add your remote repo

    (<project_virtual_env_name>)$ git remote add origin <you_remote_repo_url>

then finally pull the repo here

    (<project_virtual_env_name>)$ git pull origin master

and then install your requirements while the virtualenv still activated

    (<project_virtual_env_name>)$ pip install -r requirements.txt

(like this your server will be synced with your development environmet,
so when ever you want to make changes, just make pull to your repo)

after install your requirements, do the following commands
but before that make sure than your setting file is compatible with your server
    eg : database password and user name

then execute the following :
    (<project_virtual_env_name>)$ python manage.py makemigrations
    (<project_virtual_env_name>)$ python manage.py migrate
    (<project_virtual_env_name>)$ python manage.py createsuperuser
    # if needed create a super user
    (<project_virtual_env_name>)$ python manage.py collectstatic
    (<project_virtual_env_name>)$ sudo ufw allow 8000
    # or the port you will run your server on it
    (<project_virtual_env_name>)$ python manage.py runserver 0.0.0.0:8000

then go to your brower and check if it's live and runnung

     (<project_virtual_env_name>)$ gunicorn --bind 0.0.0.0:8000 <your_project_name>.wsgi
     (<project_virtual_env_name>)$ deactivate

Service Files for Gunicorn

    $ sudo nano /etc/systemd/system/gunicorn.service

    # i used to do it without creating .sock file
    # and let the system create
    ps : you can create let the path of .sock file anywhere

    [Unit]
    Description=gunicorn daemon
    After=network.target

    [Service]
    User=new_user_name
    Group=www-data
    WorkingDirectory=/home/<new_user_name>/<project dir name>
    ExecStart=/home/<new_user_name>/<project dir name>/<project_virtual_env_name>/bin/gunicorn \
            --access-logfile - \
            --workers 3 \
            --bind unix:/home/<new_user_name>/gunicorn.sock \
            <your_project_name>.wsgi:application

    [Install]
    WantedBy=multi-user.target

    # save and exit

    $ sudo systemctl daemon-reload
    $ sudo systemctl start gunicorn
    $ sudo systemctl enable gunicorn
    $ sudo systemctl status gunicorn
    # everything should work just fine

Configure Nginx to Proxy Pass to Gunicorn

    $ sudo nano /etc/nginx/sites-available/<any_name_could_be_fine_or_your project_name>

    server {
        listen 80;
        server_name www.you-domain.com, *.you-domain.com;
        keepalive_timeout 5;
        client_max_body_size 4G;

        location = /favicon.ico { access_log off; log_not_found off; }
        location /static/ {
            root /home/<new_user_name>/<project dir name>;
        }

        location /media/ {
            root /home/<new_user_name>/<project dir name>;
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/home/<new_user_name>/gunicorn.sock;
        }
    }

    # save and exit

    $ sudo ln -s /etc/nginx/sites-available/<any_name_could_be_fine_or_your project_name> /etc/nginx/sites-enabled

    $ sudo rm /etc/nginx/sites-enabled/default
    $ sudo nginx -t
    $ sudo systemctl restart nginx
    $ sudo ufw delete allow 8000
    $ sudo ufw allow 'Nginx Full'


# enable SSL certification with CertBot

    $ sudo apt-get update
    $ sudo apt-get install software-properties-common
    $ sudo add-apt-repository ppa:certbot/certbot
    $ sudo apt-get update
    $ sudo apt-get install python-certbot-nginx
    $ sudo certbot --nginx

    # very simple installation instruction just follow it and see what suite your needs

Setup the auto renew of the certs. Run the command below to edit the crontab file:
    $ sudo crontab -e
    # choose your preferred editor and hit Enter and past the following

    0 4 * * * /usr/bin/certbot renew --dry-run

    #save and exit

# and by reaching this phase your web application is ready to run 🚀 with SSL Certification 🔐
