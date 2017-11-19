# First steps
* First ssh
    - `ssh -i /Users/Axel/Desktop/LS-key.pem ubuntu@52.39.208.193`

# User Management
1. The SSH key submitted with the project can be used to log in as grader on the server.
    * `$ sudo adduser grader` to create a new user
    * `$ sudo nano /etc/sudoers.d/grader` to give sudo access to grader
    * Add: `grader ALL=(ALL) NOPASSWD:ALL` to the file
    * Configure the key-based authentication for grader user
    * **Generating key pair:**
        * On your machine (ie, not lightsail server): `ssh-keygen` and enter a passphrase: e.g. linuxCourse
        * Keys are stored in `/Users/Axel/.ssh/` as linuxCourse and linuxCourse.pub
    * **Installing a public key:**
        * Log on your server as ubuntu and type: `sudo mkdir /home/grader/.ssh` and then `$ sudo touch /home/grader/.ssh/authorized_keys`
        * On your machine: `$ cat .ssh/linuxCourse.pub` and copy the public key displayed
        * Back on your server: `$ sudo nano /home/grader/.ssh/authorized_keys` and paste your public key
        * Change some permissions:
        * `$ sudo chmod 700 /home/grader/.ssh`
        * `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`
        * `$ sudo chown -R grader:grader /home/grader/.ssh` to change the owner from *root* to *grader*
        * Login in as grader: `ssh -i /Users/Axel/.ssh/linuxCourse grader@52.39.208.193`
    * ** Forcing Key Based Authentication:**
        * `$ sudo nano /etc/ssh/sshd_config` and make sure the line is `PasswordAuthentification no`, if necessary edit it to no
        * `$ sudo service ssh restart` to restart the service and make changes take effect

2. You cannot log in as root remotely.
    * `$ sudo nano /etc/ssh/sshd_config` to edit the PermitRootLogin line to no
    * `$ sudo service ssh restart`

3. The grader user can run commands using sudo to inspect files that are readable only by root. - SEE ABOVE


# Security
1. SSH is hosted on non-default port.
    * `$ sudo nano /etc/ssh/sshd_config` to find the port line and edit it to: 2200
    * `$ sudo service ssh restart`
    * `ssh -i /Users/Axel/.ssh/linuxCourse -p 2200 grader@52.39.208.193` to login using port 2200

2. Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
    * `sudo ufw default deny incoming` to block all incoming requests
    * `sudo ufw default allow outgoing` to allow all outgoing requests
    * `$ sudo ufw allow 2200/tcp` to allow incoming connections for SSH (port 2200)
    * `$ sudo ufw allow 80/tcp` to allow incoming connections for HTTP (port 80)
    * `$ sudo ufw allow 123/udp` to allow incoming connections for NTP (port 123)
    * `$ sudo ufw enable`to enable the firewall

3. Key-based SSH authentication is enforced. - SEE ABOVE

4. All system packages have been updated to most recent versions.
    * `$ sudo apt-get update` to update available package lists
    * `$ sudo apt-get upgrade` to upgrade installed packages
        * if prompted install the maintainer package version
    * `$ sudo apt-get install finger` to install *finger*, a utility software to check users' status


# Application Functionality
1. The web server responds on port 80. - SEE ABOVE

2. Database server has been configured to serve data (PostgreSQL is recommended).
    * **PostgreSQL: installation & configuration**
        * `sudo apt-get install postgresql` to install PostgreSQL
        * `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` to check that no remote connections are allowed
        * `sudo su - postgres` to login as postgres
        * `psql` to access the shell
        * `CREATE DATABASE catalog;` to create the catalog Database
        * `CREATE USER catalog;` to create the catalog user
        * `ALTER ROLE catalog WITH PASSWORD 'password';` to set a password for the user catalog
        * `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;` to give the user catalog permissions to the catalog database
        * `\q` to exit the shell and `exit` to exit PostgreSQL
    * **Setting up the catalog app**
        * `sudo apt-get install git` to install git
        * `cd /var/www` to move into the www folder
        * `sudo mkdir FlaskApp` to create a flask directory
        * `cd FlaskApp` to move into the newly created directory
        * `git clone https://github.com/a-gabay/catalog.git` to clone your project
        * `sudo mv ./catalog ./FlaskApp` to rename the project into FlaskApp
        * `cd FlaskApp` to move into the directory
        * `sudo mv project.py __init__.py` to rename the name of the project.py file
        * Update the engine in __init__.py file, database_setup.py, and populate.py files : `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
        * `sudo apt-get install python-pip` to install pip
        * Install all required dependencies :
            * `sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests`
            * `sudo pip install flask packaging oauth2client redis passlib flask-httpauth`
            * `sudo apt-get -qqy install postgresql python-psycopg2`
        * `sudo python database_setup.py` to create the DB
        * `sudo pip install populate.py` to fill the DB
    * **apache2 configuration**
        * `sudo nano /etc/apache2/sites-available/FlaskApp.conf` to create and edit the FlaskApp.conf file
        * Add the following to the file
        ```
    	<VirtualHost *:80>
    		ServerName 52.39.208.193
    		ServerAdmin 52.39.208.193@admin
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
        * `sudo a2ensite FlaskApp` to enable the website
        * `cd /var/www/FlaskApp` to move into the FlaskApp Directory
        * `sudo nano flaskapp.wsgi`to create a wsgi file and add :
        ```
    	#!/usr/bin/python
    	import sys
    	import logging
    	logging.basicConfig(stream=sys.stderr)
    	sys.path.insert(0,"/var/www/FlaskApp/")

    	from FlaskApp import app as application
    	application.secret_key = 'super_secret_key'
    	```
        * `sudo service apache2 restart ` to restart appache
        * Update paths in project.py for both client secrets file to reflect the new location by adding: `/var/www/catalog/catalog/`to the path
        * change authorized redirect URLs for FB and google connect:
        go to :
            * https://console.developers.google.com,
            * https://developers.facebook.com
        * and add the following :
            * http://52.39.208.193

3. Web server has been configured to serve the Item Catalog application as a WSGI app.
    * `$ sudo apt-get install apache2` to install Apache
    * `$ sudo apt-get install libapache2-mod-wsgi` to install mod_wsgi
    * `$ sudo nano /etc/apache2/sites-enabled/000-default.conf` add the following line at the end of the `<VirtualHost *:80>` block, right before the closing `</VirtualHost>` line: `WSGIScriptAlias / /var/www/html/myapp.wsgi` (should be correctly indented)
    * `* sudo nano /var/www/html/myapp.wsgi` and add to the file:
    ```
    def application(environ, start_response):
        status = '200 OK'
        output = 'Hello Udacity!'

        response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
        start_response(status, response_headers)

        return [output]
    ```
    * Go to http://34.213.54.151/ (your lightsail ip address) to check that you get the hello Udacity! message

# URL to hosted web application
* [link to hosted web application](http://52.39.208.193/catalog)

# Documentation
- A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project.

# Sources
* [sharkwhistle's README](https://raw.githubusercontent.com/sharkwhistle/Udacity-FSND-Linux-Server-Configuration-/master/README.md)
* [iliketomatoes' README](https://github.com/iliketomatoes/linux_server_configuration/blob/master/README.md)
* [mod_wsgi (Apache) in Flask documentation](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
