Linux Server Configuration
==========================

A Flask based web application on a baseline installation of a Linux distribution on a virtual machine.

Server Details
--------------
The catalog web app can be accesed through following links:

* **Server IP:**  
  http://52.37.200.141/  
* **SSH Port:** 2200  
* **Application URL:**  
  http://ec2-52-37-200-141.us-west-2.compute.amazonaws.com/  


1. Launch a virtual machine and SSH into server
--------------------------------------------

  * Launch a  virtual machine using udacity account.
  * Download the private key and notice down the public IP address.
  * Change file rights:  
      `$ chmod 600 ~/.ssh/udacity_key.rsa`
  * SSH into the instance:  
      `$ ssh -i ~/.ssh/udacity_key.rsa root@52.37.200.141`

2. Append the hostname to the localhost
---------------------------------------

  * In /etc/hosts add  
      `127.0.0.1 localhost ip-10-20-29-232`

3. Create a new user and give it sudo permission
------------------------------------------------
  * Create a new user _grader_:  
      `$ adduser grader`
  * Give _grader_ the sudo permission
  ```
  $ echo "grader ALL=(ALL) NOPASSWD:ALL" >/etc/sudoers.d/grader
  $ chmod 0440 /etc/sudoers.d/grader
  $ su -l grader
  ```

4. Update and upgrade all the installed packages
------------------------------------------------------
  * Update the package list:  
      `$ sudo apt-get update`
  * Install newer versions:  
      `$ sudo sudo apt-get upgrade`

5. Configure SSH login for grader
----------------------------------------------------
  * Generate a SSH key pair on the local machine:  
      `$ ssh-keygen`
  * Copy the public key to _grader_ folder:  
      `$ scp -i ~/udacity_key.rsa /fs_p5.pub root@52.37.200.141:/home/grader`
  * Log in to remote machine  
      `$ ssh -i ~/udacity_key.rsa root@52.37.200.141`
  * Give _grader_ file ownership  
      `chown grader:grader /home/grader/fs_p5.pub`
  * Add this key to _authorized_ list
  ```
  $ mkdir .ssh
  $ chown grader:grader .ssh
  $ cat fs_p5.pub > .ssh/authorized_keys
  $ chmod 700 .ssh/
  $ chmod 644 .ssh/authorized_keys
  $ exit
  ```

6. Change SSH port from 22 to 2200
----------------------------------

  * Open the config file:  
    `$ vim /etc/ssh/sshd_config`
  * Change to Port 2200.
  * Change `PermitRootLogin` from `without-password` to `no`.
  * Append `AllowUsers grader`.
  * Restart the ssh server  
    `sudo service ssh restart`
  * Log out `$ exit`
  * Test to see that _root_ is not allowed anymore  
    `$ ssh -p 2200 -o "IdentitiesOnly yes" -i ~/udacity_key.rsa root@52.37.200.141`
  * Login as a _grader_  
    `$ ssh -p 2200 grader@52.37.200.141`
  * Check identity `$ whoami`

7. Configure uncomplicated firewall (UFW)
-----------------------------------------
  * Configure to allow SSH (port 2220) and HTTP (port 80)  

    ```console
    $ sudo ufw status                                                                                   
    Status: inactive
    $ sudo ufw default deny incoming
    Default incoming policy changed to 'deny'
    (be sure to update your rules accordingly)
    $ sudo ufw default allow outgoing
    Default outgoing policy changed to 'allow'
    (be sure to update your rules accordingly)
    $ sudo ufw allow 2200/tcp
    Rules updated
    Rules updated (v6)
    $ sudo ufw allow www
    Rules updated
    Rules updated (v6)
    $ sudo ufw show added
    Added user rules (see 'ufw status' for running firewall):
    ufw allow 2200/tcp
    ufw allow 80/tcp
    $ sudo ufw enable
    Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    Firewall is active and enabled on system startup
    $ sudo ufw status
    Status: active

    To                         Action      From
    --                         ------      ----
    2200/tcp                   ALLOW       Anywhere
    80/tcp                     ALLOW       Anywhere
    2200/tcp (v6)              ALLOW       Anywhere (v6)
    80/tcp (v6)                ALLOW       Anywhere (v6)

    $ exit
    logout
    Connection to 52.37.200.141 closed.
    $ ssh -i fs_p5 -p 2200 grader@52.37.200.141
    $ whoami
    grader
    ```

  * Configure to allow NTP (port 123) and UTC timezone

    ```console
    $ date
    Wed Apr 27 00:38:55 UTC 2016
    $ sudo apt-get install ntp
    $ ls /etc/init.d/ntp
    /etc/init.d/ntp
    $ sudo service ntp start
     * Starting NTP server ntpd
       ...done.
    ```

8. Install and configure Apache to serve a mod wsgi application
---------------------------------------------------------------
  * Install apache and mod wsgi  
    `$ sudo apt-get install apache2 libapache2-mod-wsgi`
  * Enable mod wsgi  
    ```
    $ sudo a2enmod wsgi
    Module wsgi already enabled
    ```
  * Test the website
    ```console
    $ curl -I http://52.37.200.141
    HTTP/1.1 200 OK
    Date: Sat, 30 Apr 2016 15:05:10 GMT
    Server: Apache/2.4.7 (Ubuntu)
    Content-Length: 2832
    Vary: Accept-Encoding
    Content-Type: text/html; charset=utf-8
    ```

9. Setup catalog project
------------------------

### 9.1 Install git and configure settings

  * Install Git:  
    `$ sudo apt-get install git`
  * Set your name, e.g. for the commits:  
    `$ git config --global user.name "NAME"`
  * Set up your email address to connect your commits to your account:      
  `$ git config --global user.email "EMAIL ADDRESS"`  

### 9.2 Setup apache for flask web app

  * Install pip installer  
    `$ sudo apt-get install python-pip`

  * Install all python requirements
    `$ sudo pip install -r requirements.txt`

  * Create a catalog directory
    `$ mkdir \var\www\catalog`
    `$ mkdir \var\www\catalog\catalog`

  * Configure and enable a virtual host
    ```console
    $ sudo nano /etc/apache2/sites-available/catalog.conf

      <VirtualHost *:80>
          ServerName PUBLIC-IP-ADDRESS
          ServerAdmin admin@PUBLIC-IP-ADDRESS
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

    $ sudo a2ensite catalog
    ```

  * Create a mod-wsgi file and restart apache
    ```console
    $ cd /var/www/catalog
    $ nano /var/www/catalog/catalog.wsgi

    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'
    ```
  * Restart apache server  
    `$ sudo service apache2 restart`

  ### 9.3 Clone git repository and make it web inaccessible

  * Clone catalog project from github  
    `$ git clone https://github.com/pmishra02138/fullstack-vm.git`

  * Move the contenets of catalog project to var folder
    `$ mv fullstack-vm/vagrant/catalog/* /var/www/catalog/catalog/`

  * Delete rest of the cloned git repository  
    `$ rm -rf fullstack-vm`

  * Make the GitHub repository inaccessible:  
      * Create and open .htaccess file:  
        `$ cd /var/www/catalog/` and `$ sudo nano .htaccess`
      * Paste in the following:  
        `RedirectMatch 404 /\.git`
