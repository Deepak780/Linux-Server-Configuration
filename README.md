# Linux-Server-Configuration

## Starting a Server
### Step 1: Create a Server on Amazon EC2
* login to [AWS Account](aws.educate.com)
* Choose EC2 and select Ubuntu with appropriate settings.
* Click on **Review and Launch** and select **Launch**.
* Edit and select **Create a new pair** and then download the file with extension '.pem'.
* Then launch the instance.
* Give the name for instance and then goto **description** section.
* In **Inbounds** section, select **Launch-wizaed-2** and add ports for SSH (2200), HTTP (80) and NTP (123) and save.

### Step 2: Secure server
```
sudo apt-get update
```
```
sudo apt-get upgrade
```

### Step 3: Change SSH port from 22 to 2200
* Edit the file: sudo vi /etc/ssh/sshd_config
Change Port number 22 to 2200
PubkeyAuthentication yes
PasswordAuthentication no
Save and exit using esc and confirm with :wq
* Restart the server: sudo service ssh restart
* To check port 2200: Working or Not
```
ssh -i linux_server_06_02_2019.pem -p 2200 ubuntu@3.87.44.0
```

### Step 4: Configure the Uncomplicated Firewall (UFW)
```
* sudo ufw status
* sudo ufw default deny incoming
* sudo ufw default allow outgoing
* sudo ufw allow 2200/tcp
* sudo ufw allow www
* sudo ufw allow 123/udp
* sudo ufw deny 22

* sudo ufw enable
```

```
sudo ufw status
```
You wii get the result:
```
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)
```

### Step 5: Give `grader` access
* Create a New User account named `grader`
`sudo adduser grader` and enter password.

* Give grader the permission to sudo.
1. Edits the sudoers file: `sudo visudo` 
2. Add a new line` grader  ALL=(ALL:ALL) ALL` to give sudo privileges to grader user after the following line:
root    ALL=(ALL:ALL) ALL
3. Save and exit using ctrl+x and confirm with Y.

4. Run su - grader, enter the password.

* Create an SSH key pair for grader using the ssh-keygen tool.
1. create .ssh folder by mkdir /home/grader/.ssh
2. su grader
3. RUN the following command 
```
sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
```
4. change Ownership 
```
chown grader.grader /home/grader/.ssh
```
5. Goto root directory using the command
```
sudo su
```
6. sudo Group add 
```
sudo su usermod -aG sudo grader
```
7. Change permissions for .ssh folder
```
chmod 0700 /home/grader/.ssh/, for authorized_keys chmod 644 authorized_keys
```
8. Check in `vi /etc/ssh/sshd_config` file 
if PermitRootLogin is set to No
9.Restart SSH: `sudo service ssh restart`
10. Check whether grader account is Working or not by RUNNING the command
```
ssh -i linux_31.pem -p 2200 grader@3.87.44.0
```
#### Configure the local timezone to UTC Logged On grader Account
TIME ZONE: 
```
sudo dpkg-reconfigure tzdata
```
Choose time zone UTC

### Instaling Apache
1. Now install apache software as grader.
```
  sudo apt-get install apache2
```
2. Enter public IP of the Amazon EC2 instance into browser to check whether Apache installed or not. If success, it displays the APACHE PAGE. Step 3. Now again install library functions of apache using the command
```
sudo apt-get install libapache2-mod-wsgi-py3
```
Step 3. Enable the mod_wsgi using the command:
```
sudo a2enmod wsgi
```
Step 4. Install some libraries of python development:
```
sudo apt-get install libpq-dev python-dev	
```
### Install and configure PostgreSQL:
1. Install postgresql as:
```
sudo apt-get install postgresql postgresql-contrib.
```
2. Change to postgresql from grader user
```
sudo su - postgres
  psql
```
3. Create User named Catalog:
```
CREATE USER catalog WITH PASSWORD 'catalog'
```
4. Alter the user:
```
ALTER USER catalog CREATEDB
```
5. Create a database using catalog owner:
```
create database catalog with owner catalog
```
6. Change to catalog database:
```
\c catalog.
```
7. Revoke all the schemas:
```
REVOKE ALL ON SCHEMA public FROM public;
```
8. Now grant all permissions to the public schemas to catalog:
```
GRANT ALL ON SCHEMA public TO catalog;
```
9. then exit using `exit` command.

### Setting up Google Oauth2 Credentials
* Login to console.developers.google.com and select a new project and name it as catalog.
* Goto credentials  and edit OAuth details(Configuration) as follows:
Javascript origin 
```
http://ip.xip.io 
```

redirect URI:
```
http://ip.xip.io/login
http://ip.xip.io/gconnect
http://ip.xip.io/callback
```
xip.io is a free DNS which will be the same as using IP address
* After download the client secrets file.

### Installing Git:
Login as grader and install git using the command:
```
sudo apt-get install git
```
### Clone and setup the Item_Catalog project from GitHub repo
* Clone from the project from /var/www directory
```
sudo git clone `https://github.com/Deepak780/catalog.git
```
* Change ownership of catalog directory using the command:
```
sudo chown -R grader:grader catalog/
```
* Change directory to `/var/www/catalog/catalog `
* Rename the mainpage.py file to __init__.py using the command
```
mv item_catalog.py __init__.py
```
* Edit the following line in all the three files __init.py, my_database.py, sample_data.py
```
engine = create_engine("sqlite:///GadgetDB.db") 
```
as
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

### Installing the virtual environment
* From /var/www/catalog/catalog directory install `pip`: 
```
sudo apt-get install python3-pip.
```
* Install the virtual environment: 
```
sudo apt-get install python-virtualenv
```
* Create the virtual environment: 
```
sudo virtualenv -p python3 venv3.
```
* Change the ownership to grader with: 
```
sudo chown -R grader:grader venv3/.
```

### Configure and Enabling New Virtual Host
* Configure by typing the following command:
```
sudo nano /etc/apache2/sites-available/catalog.conf
```
* Add the following lines:
```
<VirtualHost *:80>
    ServerName 3.87.44.0.xip.io
    ServerAlias ec2-3-87-44-0.compute-1.amazonaws.com
    ServerAdmin ubuntu@54.210.140.47
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv3/lib/python3.6/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Enable  the virtual host by using the command:
```
sudo a2ensite catalog
```
* Type the following command for restarting the apache2:
```
service apache2 reload
```

* Seting up the Flask application
* Create /var/www/catalog/catalog.wsgi file using:
```
sudo vi catalog.wsgi
```
* Add the following lines:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'supersecretkey'
```
* Restart Apache: 
```
sudo service apache2 restart.
```

From the /var/www/catalog/catalog/ directory

### Activate the virtual environment and installing dependencies: 
* Activate virtual environment using the command:
```
. venv3/bin/activate
```
* Install the following dependencies:
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2-binary
```

### Run the Project: 
1. `python my_database.py`
2. `python __init__.py'

- Deactivate the virtual environment: `deactivate`

#### Disable the default Apache site
- Disable the default Apache site: `sudo a2dissite 000-default.conf`

Then it will display as follows:

`Site 000-default disabled`
- To activate the new configuration, you need to run:
  `service apache2 reload`

* Then reload Apache: 
```
sudo service apache2 reload
```


#### Final Step
- Security Updates and package updates Try these commands:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

#### Launch the Web Application
 
 Open your browser to : (http://3.87.44.0.xip.io)
 Open your browser to : (ec2-3-87-44-0.compute-1.amazonaws.com)
 
Special Thanks to :  *PrasannaRajMallipudi* for guiding me in writing the README file for **Linux Server Configuration Project**.

The following sources have become very helpful to completing **Linux Server Configuration Project**.
* Google(https://www.howtoforge.com/tutorial/how-to-setup-linux-server-with-aws/)
* YouTube(https://youtu.be/TjVWpNZfTPE)
* [StackOverflow](https://stackoverflow.com/)
* [Udacity](https://in.udacity.com/course/configuring-linux-web-servers--ud299)
* DigitalOcean(https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
