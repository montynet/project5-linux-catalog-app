### project5-linux-catalog-app
A Ubuntu Linux server on a virtual machine to hosting a Flask web application.
This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

##i. The IP address of my public environment is 52.32.130.204 and port is 2200

## ii. The complete url of my application is http://ec2-52-32-130-204.us-west-2.compute.amazonaws.com/

## iii. The following steps were taken to complete this project along with things done

- Step 1 Get development environment from https://www.udacity.com/account#!/development_environment
- Step 2 SSH to server as explained in guide
- Step 3 Create a user call grader, add to sudoers.d, and create a key for user
  - a. adduser grader
  - b. nano /etc/sudoers.d/grader
  - c. ssh-keygen on local machine and copy key to /home/grader/.ssh/authorized_keys
- Step 4 Update the packages installed using
  - a. Using grader with sudo permissions sudo apt-get update
  - b. sudo apt-get upgrade
- Step 5 Change the ssh port from 22 to 2200
  - a. change the port using sudo nano /etc/ssh/sshd_config
  - b. restart the ssh service using sudo service ssh restart
- Step 6 Configure the ufw, ensuring this is done right to avoid getting locked out
  - a. By default the ufw is disable, deny incoming using sudo ufw default deny incoming
  - b. outgoing sudo ufw default allow outgoing
  - c. allow ports per instructions sudo ufw allow 2200/tcp
  - sudo ufw allow 80/tcp
  - sudo ufw allow 123/udp
- Step 7 Configure local time zone using
  - a. sudo dpkg-reconfigure tzdata
  - choosing 'None of the above' and then 'UTC'
  - b. set up ntp daemon using sudo apt-get install ntp
  - c. Add these server pools in sudo nano /etc/ntp.conf
    ```
    server 0.north-america.pool.ntp.org
    server 1.north-america.pool.ntp.org
    server 2.north-america.pool.ntp.org
    server 3.north-america.pool.ntp.org
    ```
- Step 8 Install and configure apache to serve a Python mod_wsgi application
  - a. sudo apt-get install apache2
  - b. sudo apt-get install python-setuptools libapache2-mod-wsgi
  - c. restart apache using sudo service apache2 restart
- Step 9 Set up git and clone project 3 application in desired directory and deploy Flask app
  - a. sudo apt-get install git
  - b. sudo apt-get install libapache2-mod-wsgi python-dev
  - c. sudo a2enmod wsgi
  - d.  create the directory for the Flask app
    cd /var/www
    sudo mkdir catalog
    cd catalog and then sudo mkdir catalog
    cd catalog and then sudo mkdir static templates
  - e. install Flask sudo apt-get install python-pip
  - f. install virtualenv sudo pip install virtualenv
  - g. make the virtual environment sudo virtual venv
  - h. change permissions using sudo chmod -R 777 venv
  - i. activate and install Flask
  - source venv/bin/activate
    pip install Flask
  - j. create virtual host config in sudo nano /etc/apache2/sites-available/catalog.conf
    ```
      <VirtualHost *:80>
          ServerName 52.32.130.204
          ServerAdmin admin@52.32.130.204
          ServerAlias HOSTNAME
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
  - k.enable host sudo a2ensite catalog
  - l. create the wsgi file sudo nano /var/www/catalog/catalog.wsgi
      ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/catalog/")
      from catalog import app as application
      application.secret_key = 'Add your secret key'
      ```
  - m. restart apache sudo service apache2 restart
  - n. clone and make .git inaccessible
  - cd /var/www/catalog
    git clone https://github.com/montynet/category-subcategory.git
    sudo mv /var/www/catalog/category-subcategory/* /var/www/catalog/catalog/
    sudo nano .htaccess
    RedirectMatch 404 /\.git
  - o. install module and packages for the app to work
    source venv/bin/activate
    pip install httplib2
    pip install requests
    sudo pip install flask-seasurf
    sudo pip install --upgrade oauth2client
    sudo pip install sqlalchemy
    sudo pip psycopg2
  - p. install postgresql and configure it
    sudo apt-get install postgresql postgresql-contrib
    change the sqlite create engine in /var/www/catalog/catalog/catalog_db_setup.py to create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')
    change the sqlite create engine in /var/www/catalog/catalog/catalog.py to create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')
    change gconnect and gdisconnect to ensure it works with werkzeug, flask and Flask-Login in /var/www/catalog/catalog/catalog.py
    change the client_secrets to include the new redirect uris
    change the client_secrets references to full paths in /var/www/catalog/catalog/catalog.py
    ```
    sudo mv /var/www/catalog/catalog/catalog.py /var/www/catalog/catalog/__init__.py
    sudo adduser catalog
    sudo su - postgres
    psql
    CREATE USER catalog WITH PASSWORD 'PW-FOR-DB';
    ALTER USER catalog CREATEDB;
    CREATE DATABASE catalog WITH OWNER catalog;
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public TO catalog;
    ```
  - q. run the db set up using python catalog_db_setup.py
  - u. in https://console.developers.google.com/project add/change the APIs & auth credentials, restart apache,
    and go to app
    sudo service apache2 restart

## iv. The following is a list of resources used to complete this project
- [Class Discussion](https://discussions.udacity.com/c/nd004-p5-linux-based-server-configuration)
- [Making oauth work](https://discussions.udacity.com/t/google-oauth2-ajax-call-not-working/31912)
- [Making oauth work p2](https://discussions.udacity.com/t/google-sign-in-oauth-error/29733/2)
- [Making oauth work p3](https://discussions.udacity.com/t/google-sign-in-problems/28191)
- [Making oauth work p4...you get the point](https://discussions.udacity.com/t/oauth2credentials-object-is-not-json-serializable/18472/4)
- [Class Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
- [Additional Resource for project](https://discussions.udacity.com/t/p5-how-i-got-through-it/15342/6)
- [Making that client secret file to be recognized](http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory)
- [Additional resource for project p2](https://discussions.udacity.com/t/project-5-resources/28343)
- [Additional resources for project p3](https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587)

## Additional configurations made after first review
- Step 1: Disable root access, log in as grader, copy root key to grader and disable root entry without password
  - 
  
`sudo cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
sudo nano /etc/ssh/sshd_config
sudo service ssh restart`

- Step 2: Install automatic updates, using the gui configuration

  - 
  `
    sudo apt-get install unattended-upgrades
    sudo dpkg-reconfigure -plow unattended-upgrades
  `

- Step 3: Configure fail to ban
install service 
  - `sudo apt-get install fail2ban`
  - follow guide [fail2ban for ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)

- Step 4: Configure automated feedback on application availability status
  - Follow guide [Munin on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps)

- Step 5: Fix the README.md
  - Using guide on [Github Markdown Basics](https://help.github.com/articles/markdown-basics/)
  - Using [live preview markdown](http://markdownlivepreview.com/)




