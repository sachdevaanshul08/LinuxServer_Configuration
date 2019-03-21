# About The Project

#####  Linux Server Configuration

This project is setup of [ItemCatalog] project on Amazon Lightsail web server.
On Amazon Lightsail account I have created one Ubuntu Instance and configured a virtual machine to deploy python enabled application.

##### About Application
This is RESTful application developed in Python and Flask, which displays Item based on category and let the authorized users perfrom CRUD operations on Items.
For Authorization we have used OAuth2.0 framework to securily loggin in Google account using google apis.

-   WebSite link :- http://ec2-13-232-197-59.ap-south-1.compute.amazonaws.com/
-   Catalog json - http://ec2-13-232-197-59.ap-south-1.compute.amazonaws.com/catalog.json
-   IP Address - 13.232.197.59
-   SSH Port : 2200

#### 1. SSH to lightsail server instance
1. Download the AWS provided default private key from the amazon GUI to your local machine.
2. Change the downloaded key file permission to `chmod 600`
3. Form you local machine's terminal type  - `ssh ubuntu@13.232.197.59 -p 22 -i ~/Downloads/name_of_the_file.pem`
4. You will be logged in successfully.

#### 2. Update the installed packages
1. Let your system know about all the latest version of softwares installed by typing - `sudo apt-get update`
2. To install the newer versions of the packages type `sudo apt-get upgrade`

#### 3. Configure SSH Port from 22 to 2200
1. To change the ssh port we have to make the changes in config file - `sudo nano /etc/ssh/sshd_config`
2. Change 22 to 2200.
3. Restart the SSH service `sudo service ssh restart`

As mentioned on project itslef on udacity -
`Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.`

Solution - (if it helps someone)
After this step make sure you do not logout from you server instance from your local terminal before following the next step, but just in case if you do- then go the lightstail amazon portal GUI and allow for `custom tcp port 2200` to let you back in 
on your Ubuntu Server instance using `ssh ubuntu@13.232.197.59 -p 2200 -i ~/Downloads/name_of_the_file.pem`

#### 4. Configure the Uncomplicated Firewall(UFW)
1. Follow the below commands one by one to configure the firebase before enabling it.
Configure to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
    ```

    sudo ufw allow 2200/tcp
    sudo ufw allow www
    sudo ufw allow 123/udp
    sudo ufw deny 22
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw enable
    sudo ufw status

    ```


#### 5. Create new user named grader
1. Logged into your virtual server machine using ssh on your terminal.
2. Create new user named grader - `sudo adduser grader`
It will ask for password (Mandatory)  and some additional information which would be optional.
3. Give grader the permission to sudo - Follow below mentiond steps -
Give grader the permission to sudo by creating a new file in sudoer's directory `sudo touch /etc/sudoers.d/grader`
4. Check if grader file was added in sudoers directory `sudo ls /etc/sudoers.d`
5. Edit the file grader to add the sudo access `sudo nano /etc/sudoers.d/grader`
    grader ALL=(ALL) NOPASSWD:ALL
    Save changes(Ctrl+X, Y, Enter)</br>

`-- To enforcing new user to connect to Virutal Server only using SSH.`
6. Create an SSH key pair for grader using the `ssh-keygen` tool
On the local machine terminal, generate key pair using ssh-keygen, it will ask for key name, pass the conventional route - `/users/your-username/.ssh/name-ofthefile`
I have passed 'grader' instead of 'name-ofthefile' & you can pass the file name of your own choice.
It will generate then generate two files grader  and grader.pub.
and in your case it will `name-ofthefile` and `name-ofthefile.pub`.
Copy the content of your 'name-ofthefile.pub' - `sudo cat name-ofthefile.pub` and copy.

7. On the contrary, on the server side i.e virtual machine:
Switch from the existing user to the newly created user that is 'grader'
`su - grader`
It will ask for password and pass the password to switch the account.
    ```

    `mkdir .ssh`
    `touch .ssh/authorized_keys`
    `nano .ssh/authorized_keys`

    ```
pass the content which you have copied in step 6 from the public file.
Now, change the access level of the directory and file as well
    `chmod 700 .ssh`
    `chmod 644 .ssh/authorized_keys`

#### 6. Prerequisites
  SSH to the virtual server from your terminal.

* Configure the local timezone to UTC
* Check current timezone setting `sudo cat /etc/timezone`
* install and configure Apache to serve a Python mod_wsgi application.
If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: `sudo apt-get install libapache2-mod-wsgi-py3`
* Once done http://13.232.197.59 serves the default apache page
* Install and configure PostgreSQL:
 `sudo apt-get install postgresql`
Do not allow remote connections
 Whether remote connections are allowed or not, it is configured pg_hba.conf file. The default config only allow connections from localhost(127.0.0.1)

   ```
    sudo ls /etc/postgresql/9.5/main
    sudo cat /etc/postgresql/9.5/main/pg_hba.conf
    sudo service apache2 restart
   ```

* Create a new database user named catalog that has limited permissions to catalog application database

    ```
    sudo su - postgres
    psql
    CREATE USER catalog WITH PASSWORD 'catalog';
    \du
    CREATE DATABASE catalog;
    \l
    ```


#### 7 - Deploy the Item Catalog Python based app on our Server and install the required modules
* Create the FlaskApp, cloning the git repository
    ```
    cd /var/www
    sudo mkdir FlaskApp
    cd FlaskApp
    sudo git clone https://github.com/sachdevaanshul08/ItemCatalog
    ```
* Go to [HCIData] and get the hostname for your IP address since while setting up OAUTH2.o we would be needing a DNS name which refers to our instance IP.
in my case it is :- http://ec2-13-232-197-59.ap-south-1.compute.amazonaws.com/

* Make changes for the app in Google API Console, URL and redirect URIs. Edit the secret files on server to reflect the changes if any.
    -  To change the file `sudo nano path-to-clients_secrets.json`
* Edit the python files to reflect the correct postgresql database connection - username, password, DBname

        `sudo nano views.py`
        `sudo nano models.py`
        `sudo nano insert_data.py`

    change engine = create_engine('sqlite:///itemcatalog.db') to engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
* Rename views.py to __init__.py using `sudo mv views.py __init__.py`
* Change the file references for client_secrets in the python file views.py 
CLIENT_ID = json.loads(open(r’/var/www/FlaskApp/ItemCatalog/client_secrets.json’, ‘r’).read())[‘web’][‘client_id’]</br>

#### 8. Install Flask and other required packages
    sudo apt-get install python-pip
    sudo pip install virtualenv
    sudo virtualenv venv
    source venv/bin/activate
    sudo pip install Flask
    pip install httplib2
    sudo apt-get install python-oauth2client
    sudo apt-get install python-requests
    sudo apt-get install python-requests
    sudo apt-get install  python-sqlalchemy
    sudo apt-get python-psycopg2
    
    If you are using python3 then there would be slight changes in the dependencies -
    
    sudo apt-get install python-pip
    sudo pip install virtualenv
    sudo virtualenv venv
    source venv/bin/activate
    sudo pip install Flask
    pip install httplib2
    sudo apt-get install python3-oauth2client
    sudo apt-get install python3-requests
    sudo apt-get install python-requests
    sudo apt-get install  python3-sqlalchemy
    sudo apt-get python3-psycopg2
   

#### 9. Configure Apache to server mod_wsgi app and Enable the FlaskApp

    sudo nano /etc/apache2/sites-available/FlaskApp.conf


       <VirtualHost *:80>
        ServerName ec2-13-232-197-59.ap-south-1.compute.amazonaws.com (HOST NAME)
        ServerAdmin anshul_sachdeva@yahoo.com
        ServerAlias 13.232.197.59
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/ItemCatalog/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        </VirtualHost>



    * Enable the virtual host
     sudo a2ensite FlaskApp

#### Create the .wsgi File

`cd /var/www/FlaskApp`
`sudo nano flaskapp.wsgi`

Add the following lines of code to the flaskapp.wsgi file

#!/usr/bin/python


        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/FlaskApp/")

        from ItemCatalog import app as application
        application.secret_key = 'My secret key'

        #### Restart Apache
        `sudo service apache2 restart`



#### 10.  Add tables and insert data in respective tables 
Run database setup file: `python /var/www/FlaskApp/ItemCatalog/models.py`</br>
Run the file: `python insert_data.py`</br>
Run the file: `python __init__.py`
Hit the browser :- `http://ec2-13-232-197-59.ap-south-1.compute.amazonaws.com/`


#### ERRORS Finding -- MOST IMPORTANT TO REACH TO THE FINAL GOAL
Check for any errors: 
`sudo cat /var/log/apache2/error.log`


* Resources
Udacity forums</br>
Udacity Knowledge portal</br>
Udacity Student Hub</br>
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps</br>
https://www.postgresql.org/docs/current/static/sql-createuser.html</br>


[HCIData]:<https://www.hcidata.info/host2ip.cgi>
[ItemCatalog]:<https://github.com/sachdevaanshul08/ItemCatalog>