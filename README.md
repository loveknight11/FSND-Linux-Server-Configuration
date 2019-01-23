# FSWD Linux Configuration Project
* Server ip address : 35.177.139.65
* SSH Port : 2200
* Server url : ec2-35-177-139-65.eu-west-2.compute.amazonaws.com [1]
## Steps used to configure server :
### 1. update the system : 
```
sudo apt-get update
sudo apt-get upgrade
```
### 2. Add user and allow sudo. 
* Add grader user : `sudo adduser grader`
* Grant grader sudo permission : `sudo usermod –aG sudo grader` [2]
* Login as grader : `su – grader`
### 3. Allow login using public/private keys
* create `.ssh` directory : `mkdir .ssh`
* create `authorized_keys` file in `.ssh` folder :``nano .ssh/authorized_keys`
* copy local key to `authorized_keys` file
* change permissions for `.ssh` folder and `authorized_keys` file
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
* Now I can log to server using private key : `ssh grader@35.177.139.65 -p 22 -i path-to-key`
### 4. Change `ssh` port to 2200 [3]
* Edit `sshd_config` file to allow ssh through port 2200 : `44-	sudo nano /etc/ssh/sshd_config`
* Replace port 22 with 2200
* Restart ssh service : `service sshd restart`
### 5. Configure firewall
* Deny all incoming requests : `sudo ufw default deny incoming`
* Allow all outgoing requests : `sudo ufw default allow outgoing`
* Allow ssh through default port : `sudo ufw allow ssh`
* Allow ssh through 2200 port : `sudo ufw allow 2200/tcp`
* Allow HTTP : `sudo ufw allow www`
* Allow NTP : `sudo ufw allow ntp`
* Enable firewall : `sudo ufw enable`
* Check firewall status : `sudo ufw status`
* Allow ports 123 and 2200 in lightsail firewall.
### 5. Install and configure apache2
* Install apache2 server : `sudo apt-get install apache2`
* Install mod_wsgi : `sudo apt-get install libapache2-mod-wsgi python-dev`
* Enable mog_wsgi : `sudo a2enmod wsgi 
`
* restart apache2 service : `sudo service apache2 restart`
### 6. Create app on server [4]
* Create app folders in `/var/www` path
```
cd /var/www
sudo mkdir AlhusinyApp
cd AlhussinyApp
sudo mkdir AlhusinyApp
cd AlhussinyApp
sudo nano __init__.py
```
* Add test Code to be sure the server is running properly
* Install pip : `sudo apt-get install python-pip`
* Install virutalenv : `sudo pip install virtualenv`
* Create virtualenv venv : `sudo virtualenv venv`
* Activate created virtualenv : `source venv/bin/activate`
* Install flask : `sudo pip install flask`
* Try to run the test code to see if it works : `sudo python __init__.py`
* Deactivate created virtualenv : `deactivate`
* Configure virtual host : `sudo nano /etc/apache2/sites-available/AlhussinyApp.conf`
* Add these lines :
```
<VirtualHost *:80>
    ServerName alhussiny.com
    ServerAdmin alhussiny@alhussiny.com
    WSGIScriptAlias / /var/www/AlhussinyApp/alhussinyapp.wsgi
    <Directory /var/www/AlhussinyApp/AlhussinyApp/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/AlhussinyApp/AlhussinyApp/static
    <Directory /var/www/AlhussinyApp/AlhussinyApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
* Enable the virtual host: `sudo a2ensite AlhussinyApp`
* Create the .wsgi file
```
cd /var/www/AlhussinyApp
sudo nano alhussinyapp.wsgi
```
* Add these lines 
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/AlhusinyApp/")

from AlhussinyApp import app as application
application.secret_key = 'Add your secret key'
```
* disable default apache2 webpage : `sudo a2dissite 000-default.conf`
* Restart apache2 service : `sudo service apache2 restart`
* Clone repo to AlhussinyApp folder
* copy required files to the App directory and delete the cloned folder
* install requirements : `sudo pip install –r requirements.txt`
* Create database file : `sudo python models.py`
* After testing site I needed to change some permissions for the database and upload folder to work properly.
### 7. Configure the local timezone to UTC [5]
* `sudo dpkg-reconfigure tzdata`
* Choose `None of the above`
* Choose `UTC`
### 8. Get google login to work [6]
* Edit apache configuration file : `sudo nano /etc/apache2/sites-available/catalog.conf`
* Add this line under ServerAdmin : `ServerAlias HOSTNAME ec2-35-177-139-65.eu-west-2.compute.amazonaws.com`
* Enable the virtual host : `sudo a2ensite AlhussinyApp`
* Add the `URL` in the authorized domains in google developers console

## References
[1] https://ipinfo.info/html/ip_checker.php
[2] https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
[3] https://www.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306
[4] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
[5] https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
[6] https://github.com/stueken/FSND-P5_Linux-Server-Configuration