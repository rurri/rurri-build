---
title: "How to Setup Drupal 7 on Fedora 16"
posted: 2012-02-1T00:00
tags: [drupal, howto, rackspace, technical, video]
vimeo_video: 69773572
track: Technical
---
Setting up a new installation of Drupal 7 is really quite easy on Fedora.  All the software you need is available with a simple yum install command, so it is pretty much just a matter of installing the necessary services, and downloading the code.  This video goes through setting up a brand new Rackspace cloud server running Fedora 16 so that it will serve drupal pages.

## Installing Services

First things first.  We need a copy of mysql, apache, php, and some sort of text editor.  I also install nmap so I can check open ports, and also a copy of git for version control.  This can all be done with one command:

`yum install emacs nmap git apache mysql-server httpd php php-mysql`

It turns out there are a few depencies I forgot that you will also need.  So the following line will also need to be run:

`yum install php-dom php-gd php-mbstring`

##Download Source##

Now that the services are installed, we need to download the latest version of drupal.  This can be found at drupal.org.  Right click on the tar.gz version, copy to the clipboard, and use it in the command list.  At the time of this writing, the latest version was drupal 7.10.

I place my drupal install in a directory called /var/www/drupal7

```
cd /var/www
wget "http://ftp.drupal.org/files/projects/drupal-7.10.tar.gz"
gzip -d drupal-7.10.tar.gz
tar -xvf drupal-7.10.tar
mv drupal-7.10 drupal7
```

## Configure Apache

Now that the source has been installed, we need to point apache to this directory.  I like to always use the Virtual Host elements in apache to configure this so that adding other hostnames later is very simple.  In my configuration I created a new file at /etc/httpd/conf.d called vhosts.conf.  Apache will read in all files in this folder automatically so just creating this file here will cause apache to load and read the config file.  Here is how my /etc/httpd/conf.d/vhosts.conf looks:

```
<VirtualHost *:80>
   ServerAdmin admin@rurri.com
   DocumentRoot "/var/www/drupal7"
   ServerName rurri.com
   ServerAlias demo1.rurri.com
   ErrorLog "/var/log/httpd/drupal7-error_log"
   CustomLog "/var/log/httpd/drupal7-access_log" common



   <Directory /var/www/drupal7>
        Options Indexes MultiViews
        AllowOverride All
        Order allow,deny
        Allow from all
           RewriteEngine on
           RewriteBase /
           RewriteCond %{REQUEST_FILENAME} !-f
           RewriteCond %{REQUEST_FILENAME} !-d
           RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
  </Directory>
</VirtualHost>
```

##Configure IpTables##

By default on the new server, iptables will only allow connections on port 22 for SSH.  We also need to open port 80 for HTTP.  This is done by simply adding a new line to the iptables configuration file.  I added the following line directly after the line refering to port 22 to my /etc/sysconfig/iptables:

`-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT`

## Configure PHP

The only thing in PHP that needs to be configured that is not ready out of the box is the pdo extenstion.  Just uncomment the following line in /etc/php.ini:

`pdo_odbc.connection_pooling=strict`

##Configure Drupal##

Drupal will do most of its own configuration through the web interface, but it does need a settings.php file that it can write to and a files folder that it can write to before it can install itself. I got it ready with the following commands:

```
cd /var/www/drupal7/sites/default/
cp default.settings.php settings.php
chmod 777 settings.php
mkdir files
chown apache:apache files
```

Once drupal is setup you will want to modify the permissions of your settings.php file to be read only:

`chmod 555 settings.php`

##Start and Enable Services##

Next we want to turn on mysql and apache, and set them up so that they always start on startup.  Lastly we also need to restart the iptables service since we did modify the file earlier to allow port 80 connections.

```
systemctl enable mysqld.service
systemctl enable httpd.service
systemctl  start mysqld.service
systemctl  start httpd.service
systemctl restart iptables.service
```

## Create database

Last thing we need to do is just to create a database.  I called mine rurri7.

`mysqladmin create rurri7`

Install through Browser

Thats it for the server configuration.  The only thing left to do is to configure drupal through your web browser.  Go to the IP Address or hostname of the machine you just created, and it should ask you if you want to install drupal.  Go through the steps and give the information as it asks for it.  For database, leave as localhost, username as root, and password as blank.

##Hardening the Server##

So you are all done installing drupal, and now wondering about how to secure your installation.  The two most common complaints of the way I have installed drupal are:

I did everything as root
I did not password protect the mysql root account

Fixing them is trivial, and can be done by adding a password to the mysql server root user and then creating a seperate user for the drupal database.  Once this user is created, these settings can be changed after the fact in the drupal config file at: /var/www/drupal7/sites/default/settings.php.

Indeed security should always be a multi-level approach and if you want to look at more steps and ways to do it, an excellent guide is available from Imminent Web Services.