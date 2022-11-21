# Make_Your_First_Django_Deploy


## Creating a Google Claude Plataform (GCP) account

### Google Cloud Plataform :desktop_computer:
Go to: Compute Engine > VM instances > Create account as your need
### The following steps will be with the boot disk in ubuntu 18.04 LTS

![print1](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print1.png)

* In Firewall allow HTTP traffic and HTTPS traffic <br />
* Create


## Create SSH keys

### Google Cloude Plataform :desktop_computer:
* Metadata > SSH KEYS

### Linux
```
ssh-keygen -f ~/.ssh/key_name -t rsa -b 4096
```
* Enter with you passphrase
```
cat ~/.ssh/key_name.pub
```
* Copy and past on GCP - SSH KEYS
* Open the server ssh User@External IP
* Are you sure you want continu connecting (yes/no)? yes


### Windows :desktop_computer:
#### Download PuTTY - https://www.putty.org/ (When you do dawnload it installs 2 app, PuTTY and PuTTYgen)
* Open PuTTYgen > Generate  (you need to keep moving your cursor mouse)
* In Key comment add you User
* Copy and past on GCP - SSH KEYS
* In PuTTYgen click in Save private key
* GCP > VM instances > (copy)External IP
* Open PuTTY > In host Name put your User@External IP
* In Saved Sessions put GOOGLE SERVER and Save

![print2](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print2.png)

* Will be generated an app, when you access will open the server

### Google Cloude Plataform :desktop_computer:
* VM instances > More actions > View network details

![print3](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print3.png)

* IP addresses > RESERVE (reseve a static ip) > Put you VM instance name > RESERVE

![print4](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print4.png)

#### Close the server if you aren't using 

![print5](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print5.png)


## Preparing Files

### PyCharm :desktop_computer:
#### Collecting static files
* In settings.py add: 
```
STATIC_ROOT = os.path.join('static')
```
* In Terminal:
```
python manage.py collectstatic
```


## Preparing Server

### Linux :desktop_computer:
```
ssh User@External IP
```

### Windows (PuTTY) :desktop_computer:
* Install Libraries

```
sudo apt update -y
sudo apt upgrade -y
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3-pip python3.10 python3.10-dev python3.10-venv gcc default-libmysqlclient-dev libssl-dev nginx curl
python3.10 -m pip install --upgrade pip setuptools wheel --user
export PATH="/home/$USER/.local/bin:$PATH"
python3.10 -m pip install pipenv --user
```
#### Create directory :exclamation:(in place of contactlist put the name of your project):exclamation:
```
mkdir contactlist
```
* Open directory
```
cd contactlist
```
#### Create an virtual environment in contactlist directory
```
python3.10 -m venv venv
```
```
source venv/bin/activate (To exit the virtual environment just type deactivate)
python3.10 -m pip install django gunicorn pillow
```


## Moving files to Server

### Windows :desktop_computer:
#### Download FileZilla (https://filezilla-project.org/download.php?type=client)
* FileZilla > Host: sftp://User@External IP

![print6](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print6.png)

* In the right area look for your server folder
* In the left area look for the project folder on your computer
#### Now you can close FileZilla

### PuTTY :desktop_computer:
#### With your venv activate
* Testing gunicorn: 
```
gunicorn --bind 0.0.0.0:8000 contact_list.wsgi
```
#### Warning! The name that should be used instead of contact_list is the one present in wsgi.py

![print7](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print7.png)
 
* Ctrl + C to stop the test
#### Create file
```
sudo nano /etc/systemd/system/gunicorn.socket
```
#### Paste (without editing)
```
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```
#### To save use the following command: Ctrl+O > Enter > Ctrl+X
#### Create another file
```
sudo nano /etc/systemd/system/gunicorn.service
```
#### Edit before pasting
```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

	[Service]
	User=#########USER############
	Group=www-data
	WorkingDirectory=/home/#########USER############/#########PROJECT############
	ExecStart=/home/#########USER############/#########PROJECT############/venv/bin/gunicorn \
	          --access-logfile - \
	          --workers 3 \
	          --bind unix:/run/gunicorn.sock \
	          #########PROJECT############.wsgi:application

	[Install]
	WantedBy=multi-user.target
```

	![print8](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print8.png)
#### To save use the following command: 
* Ctrn+O > Enter > Ctrl+X
#### Activating
```
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```
#### If you have some error try: sudo systemctl enable gunicorn.service
#### Check-system
```
sudo systemctl status gunicorn.socket
sudo systemctl status gunicorn (if is dead try the next command, you will try this again)
curl --unix-socket /run/gunicorn.sock localhost
sudo systemctl status gunicorn
```
#### Create another file

```
sudo nano /etc/nginx/sites-enabled/sitedjango
```

#### Configurating the nginx server block 
* Edit before pasting

```
server {
	    listen 80;
	    server_name localhost;
	
	    location = /favicon.ico { access_log off; log_not_found off; }
	    
	    location /static/ {
	        root /home/luiswolski/contactlist;
	    }

	    location /media {
	        alias /home/luiswolski/contactlist/media/;
	    }

	    location / {
	        include proxy_params;
	        proxy_pass http://unix:/run/gunicorn.sock;
	    }
	}
```

#### Start nginx and unicorn

```
sudo rm /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
sudo systemctl restart gunicorn
```
#### Now you have that
![print9](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print9.png)
#### Now you will edit the setting.py
```
nano contactlist/contact_list/settings.py
```
* Put you ip in ALLOWED_HOSTS

![print10](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print10.png)

#### To save use the following command:
* Ctrn+O > Enter > Ctrl+X
```
sudo systemctl restart gunicorn
```

# :tada: CONGRATUIOLATIONS, YOUR  SITE IS ONLINE :tada:

## Let's change the Database

### How to change your SQLite3 databse to MySQL(MariaDB)
* First install mysql/mariadb
```
cd your_project/your_project
sudo apt-get update
sudo apt-get install mariadb-server python3.10-dev libmysqlclient-dev	
```
* If does't work try this:
```
sudo apt-get -f install
sudo apt-get update
sudo apt-get -f install
sudo dpkg --configure -a
sudo apt-get clean
```
* And try again
```
sudo apt-get install mariadb-server python3.10-dev libmysqlclient-dev	
```
-If work you will have that
![print11](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print11.png)
* User cursopython, password curs0Pyth0n@169, bd sitedjango
```
sudo mysql -u root
USE mysql;
UPDATE user SET plugin='' WHERE User='root';
CREATE DATABASE sitedjango CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER 'cursopython'@'%' IDENTIFIED BY 'curs0Pyth0n@169';
GRANT ALL PRIVILEGES ON sitedjango.* TO 'cursopython'@'%';
FLUSH PRIVILEGES; Exit;	
```
#### Go to venv
```
source /home/$USER/contactlist/venv/bin/activate
pip install mysqlclient
```
#### Migrate
```
cd /home/$USER/contactlist/
python manage.py dumpdata > db.json
```
#### Configurate MySQL / PostgreSQL.
```
nano /home/$USER/contactlist/contact_list/settings.py
```
![print12](https://github.com/luiswolski/Make_Your_First_Django_Deploy/blob/main/Prints/print12.png)
#### To save use the following command: 
* Ctrn+O > Enter > Ctrl+X
```
cd /home/$USER/contactlist/
python manage.py migrate
python manage.py shell
```

#### Add in shell
```
from django.contrib.contenttypes.models import ContentType
ContentType.objects.all().delete()
quit()
```

```
python manage.py loaddata db.json
sudo systemctl restart nginx && sudo systemctl restart gunicorn
```
## :warning:IF EVERYTHING WORKS:warning:
```
rm db*
```
