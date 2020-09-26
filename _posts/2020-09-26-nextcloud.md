---
title: Self-hosted low-cost cloud solution with Nextcloud on Ubuntu Server 20.04.1 LTS on Raspberry Pi 4
---

Tons and tons of personal data is being uploaded, stored, used (misused) on daily basis by organizations and companies around the globe. With looming ban on TikTok over privacy concern and earlier accounts of similar concern with other companies like Facebook, I wondered whether companies will take care of my personal data while monetizing their business models which is obvious for their survival.

***"When something online is free, you're not the customer, you're the product"* - Jonathan Zittrain**

While it is not practically possible (unless you go off the grid) to completely avoid sharing your data with tech companies, one can at least minimize it. Person's data is his own responsibility too and it's up to us how much we are willing to share with the world.

I decided to take things in my own hand. So I bought Raspberry Pi 4 and ran Ubuntu Server 20.04.1 on it with goal of setting up _Nextcloud_ instance as personal cloud for my data.

Nextcloud is self-hosted productivity platform that puts your data under your control. It can store documents, calendar, contacts and photos on a server at home, at one of their providers or in a data center you trust.

This guide shows how I installed Nextcloud on Ubuntu Linux server 20.04.1 with Apache as the web server and MariaDB as database software. For installing Ubuntu on Raspberry Pi 4, you can follow this [tutorial](https://ubuntu.com/tutorials/how-to-install-ubuntu-on-your-raspberry-pi#1-overview) on Ubuntu's official website.

### INSTALL LAMP STACK IN UBUNTU (LINUX, APACHE, MARIA DB, PHP)

#### 1. Install LAMP Stack

`$ sudo apt update && sudo apt upgrade`

`$ sudo apt-get install apache2 mariadb-server libapache2-mod-php7.4 php7.4-gd php7.4-json php7.4-mysql php7.4-curl php7.4-mbstring php7.4-intl php-imagick php7.4-xml php7.4-zip`

**Note: Update the version of PHP for current applicable version. At the time of writing these instructions PHP 7.4 version was applicable.**

#### 2. Check if services are started and enabled

`$ systemctl status apache2`\
`$ systemctl status mariadb`\
`$ systemctl is-enabled apache2`\
`$ systemctl is-enabled mariadb`

If services are not started for any reason, start and enable them

`$ sudo systemctl start apache2`\
`$ sudo systemctl start mariadb`\
`$ sudo systemctl enable apache2`\
`$ sudo systemctl enable mariadb`

#### 3. Secure Maria DB server installation

`$ sudo mysql_secure_installation`

Then answer the following questions when prompted (set a strong and secure root password):

`Enter current password for root (enter for none): enter`\
`Set root password? [Y/n] y`\
`Remove anonymous users? [Y/n] y`\
`Disallow root login remotely? [Y/n] y`\
`Remove test database and access to it? [Y/n] y`\
`Reload privilege tables now? [Y/n] y`

### INSTALL NEXTCLOUD IN UBUNTU

#### 1. Create database and database user for Nextcloud

Login to Maria DB server to access the MySQL shell:

`$ sudo mysql -u root -p`

Run following sql commands:

`MariaDB [(none)]> CREATE DATABASE nextcloud;`\
`MariaDB [(none)]> CREATE USER ncadmin@localhost IDENTIFIED BY 'YOUR_PASSWORD';`\
`MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO ncadmin@localhost IDENTIFIED BY 'YOUR_PASSWORD';`\
`MariaDB [(none)]> FLUSH PRIVILEGES;`\
`MariaDB [(none)]> EXIT;`

#### 2. Download Nextcloud server with wget

`$ sudo wget -c https://download.nextcloud.com/server/releases/nextcloud-18.0.0.zip`

**Note: Update the release version with current one. At the time of writing these instructions version 18.0.0 was applicable. You can check current version at https://www.nextcloud.com/install**

#### 3. Setup nextcloud directory

Extract the archive contents:\
`$ sudo unzip nextcloud-18.0.0.zip`

Copy the extracted nextcloud directory/folder into your web server’s document root:\
`$ sudo cp -r nextcloud /var/www/html/`

Set appropriate ownership on the nextcloud directory:\
`$ sudo chown -R www-data:www-data /var/www/html/nextcloud`

### CONFIGURE APACHE TO SERVE NEXTCLOUD

#### 1. Create an Apache configuration file for Nextcloud under the /etc/apache2/sites-available directory.

`$ sudo nano /etc/apache2/sites-available/nextcloud.conf`

#### 2. Copy paste below content (Replace /var/www/html/nextcloud/ if your installation directory is different)

```
Alias /nextcloud "/var/www/html/nextcloud/"

<Directory /var/www/html/nextcloud/>
  Require all granted
  Options FollowSymlinks MultiViews
  AllowOverride All

  <IfModule mod_dav.c>
    Dav off
  </IfModule>

  SetEnv HOME /var/www//html/nextcloud
  SetEnv HTTP_HOME /var/www/html/nextcloud
</Directory>
```

#### 3. Enable the newly created nextcloud site and other Apache modules

`$ sudo a2ensite nextcloud.conf`\
`$ sudo a2enmod rewrite`\
`$ sudo a2enmod headers`\
`$ sudo a2enmod env`\
`$ sudo a2enmod dir`\
`$ sudo a2enmod mime`

#### 4. Restart Apache 2 service

`$ sudo systemctl restart apache2`

**IMPORTANT: Restart/reboot device to allow IP binding for MariaDB & Apache**

`$ sudo reboot`

### COMPLETE NEXTCLOUD INSTALLATION VIA GRAPHICAL USER INTERFACE

#### 1. Open your browser and point it to the following address:

`http://IP_ADDRESS_OF_NEXTCLOUD_SERVER/nextcloud`

#### 2. Fill in following information

**Create nextcloud admin account**\
Username:\
Password:

**Data Folder:**\
Path of data folder (/var/www/html/nextcloud/)

**Note: Path can be changed however it must be same as mentioned in nextcloud.conf file**

**Configure database:**\
Maria DB Database admin username: ncadmin\
Maria DB database admin password: ******\
Maria DB database name: nextcloud\
Maria DB server address: localhost

Nextcloud will update required configuration files, installation will complete and welcome page will be displayed.
