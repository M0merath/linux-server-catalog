LINUX SERVER CONFIGURATION
==========================
# About this file
This README was written by Kevin Lanzing for the Udacity Linux Server Configuration project. It contains important information for the grader to access the AWS server, item catalog, and hosted website.

# Essential Info (for grader)
- **My IP Address**: 18.219.96.221
- **SSH Port**: 2200
- **My URL**: http://ec2-18-219-96-221.us-east-2.compute.amazonaws.com/
- **Grader user is**: grader
- **Grader password is**: grader
- **Catalog database is**: catalog
- **Catalog password is**: catalog

# Software I used
- Git
- Git Bash
- Vagrant
- Amazon Lightsail
- Python
- Markdown (for this file)

# Third-party resources I used
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- http://www.nmonitoring.com/ip-to-domain-name.html?ip=18.219.96.221&pingsub=1&ln=en (to convert my IP into a web address)

# My Process
### Get your server.
1. Register for a Lightsail server at lightsail.aws.amazon.com. You may need to create an Amazon Web Services (AWS) account.
2. Once logged in, press the button labeled "Create an instance". Choose to install "OS Only" with Ubuntu as the OS. Choose the lowest ($5/month) tier of instance plan. Give this instance a name if you wish. I used the default "Ubuntu-512MB-Ohio-1". Wait a few minutes for this instance to start.
3. Once started, this instance will display an IP address. Mine was "18.219.96.221". Use this address in the Vagrant terminal to SSH to the server. The command is: `ssh ubuntu@18.219.96.221`.

### Secure your server.
4. Update all packages with `apt-get update` and then `apt-get upgrade`.
4. Allow ssh with Uncomplicated FireWall (UFW): `sudo ufw allow ssh`. 
5. Allow access to ports 2200 (SSH), 80 (HTTP), and 123 (NTP) with the following command: `sudo ufw allow 2200 80 123`.
6. Allow www with `sudo ufw allow www`.
7. Turn on UFW with `sudo ufw enable`.

### Give grader access
9. Create a new user 'grader' with `sudo adduser grader`. Ubuntu will prompt you to create a password for 'grader'. I used password `grader`, and skipped all other prompts (press ENTER).
9. For user 'grader' to have 'sudo' access, this user must appear in 'sudoers.d'. While logged in to the server as the 'root' or 'ubuntu' user, type `sudo cp /etc/sudoers.d/vagrant etc/sudoers.d/grader`.
10. Now access the 'grader' file you created with `sudo nano /etc/sudoers.d/grader`. Change 'ubuntu' to 'grader'. Save and exit the Nano tool. 'grader' should now have 'sudo' access!
10. `exit` the server and return to your Vagrant terminal. Generate an SSH key locally with `ssh-keygen`. You will be prompted to enter a filename for this key. I used 'grader_key'. A second prompt will ask for a password. I used 'grader'.
11. Use `cat .ssh/grader_key.pub` to reveal your public key. Copy the contents.
12. Return to your server as user 'grader' with `ssh grader@18.219.96.221 -p 2200` and password 'grader'. Use `nano .ssh/authorized_keys` and paste your public key to a single line of this file. Save and exit.
13. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` to further secure your public key.
14. From now on, access the server as grader with `ssh grader@18.219.96.221 -p 2200 -i ~/.ssh/grader_key` and password 'grader'. You may wish to test this now.

### Prepare to deploy your project.
17. Configure the local timezone to UTC with `sudo dpkg-reconfigure tzdata`. Select 'UTC'.
12. Install Apache with `sudo apt-get install apache2`.
13. Install mod_wsgi with `sudo apt-get install python-setuptools libapache2-mod-wsgi`.
14. (Re)start Apache with `sudo service apache2 restart`.
15. Install PostgreSQL with `sudo apt-get install postgresql`.
16. Login as user 'postgres' with `sudo su - postgres`.
17. Login to postgreSQL shell with `psql`.
18. Create a new database in PSQL named 'catalog' with `create database catalog;`
19. Create a new user named 'catalog' with `create user catalog;`
20. Set a password for user 'catalog' with `alter role catalog with password 'catalog';`
21. Grant user 'catalog' permission to database 'catalog' with `grant all privileges on database catalog to catalog;`
22. Quit PSQL with `\q`. Exit 'postgres' with `exit`.
29. Install 'git' with `sudo apt-get install git`.

### Deploy the Item Catalog project.
30. Use `cd /var/www` to navigate to the directory where you will place your application.
16. Create application directory with `sudo mkdir catalog`. Use `cd catalog` to enter the folder you just created.
17. Clone the Item Catalog with `git clone https://github.com/M0merath/Catalog.git`.
18. Rename `catalog.py` to `__init__.py` with `sudo mv catalog.py __init__.py`.
19. Edit 'databade_setup.py' with `sudo nano database_setup.py`. Change 'engine = create_engine' line to: `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`.
20. Install pip with `sudo apt-get install python-pip`. Using pip, install dependencies with `sudo pip install -r requirements.txt`.
21. Install `psycopg2` with `sudo apt-get -qqy install postgresql python-psycopg2`.
22. Initialize your database with `sudo python database_setup.py`.
23. Create and configure the virtual host with `sudo nano /etc/apache2/sites-available/catalog.conf`.
24. Copy the following lines of code to 'catalog.conf' to configure:
```
<VirtualHost *:80>
    ServerName http://ec2-18-219-96-221.us-east-2.compute.amazonaws.com/
    ServerAdmin klanzing@gmail.com
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
25. Enable the virtual host with `sudo a2ensite catalog`.
26. Create the .wsgi file with `sudo nano catalog.wsgi`. Copy the following lines of code into the file, and save:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```
25. Restart Apache with `sudo service apache2 restart`.
26. Navigate to http://ec2-18-219-96-221.us-east-2.compute.amazonaws.com/ with your favorite browser. If all went well, the catalog project 'Gastronaut.com' should load. Congratulations!