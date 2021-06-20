# How to install Ministra TV Platform in Ubuntu 20.04
## Installation & Configuration Ministra TV Platform on Ubuntu 20.04

[About Ministra TV Platform](https://www.infomir.eu/eng/solutions/ministra-tv-platform/)

1. Send Request For a Latest version of Ministra tv platform from [About](https://www.infomir.eu/eng/solutions/ministra-tv-platform/) page and download the zip file. 
> In my case i just got [ministra-5.6.1.zip](http://download.middleware-stalker.com/downloads/<SECRET_TOKEN>/ministra-5.6.1.zip). We'll use it in the future example.

2. Log in to your server as a `root` user.

3. Install and configure the required services and packages:
```bash
# Ubuntu 20.04 has php7.2+ as the default version of PHP but ministry 5.6.1 can work with PHP version <= 7.1 So, we'll use php7.0.
# Add repository for php7.0
apt install software-properties-common & add-apt-repository ppa:ondrej/php 
apt update & apt upgrade -y 
apt remove php* -y 
# Install required packages
apt -y install apache2 nginx memcached curl mysql-server php7.0 php7.0-mysql php7.0-memcached  php7.0-curl php-pear php7.0-xml php7.0-mcrypt php7.0-zip php7.0-sqlite3 php7.0-imagick  php7.0-soap php7.0-intl php7.0-gettext php7.0-tidy php7.0-geoip nodejs systemd-sysv 
# Set php7.0 as default PHP.
update-alternatives --set php /usr/bin/php7.0 
a2dismod php7.* 
a2enmod php7.0


# Download Ministra
cd /var/www/html/
wget http://download.middleware-stalker.com/downloads/<SECRET_TOKEN>/ministra-5.6.1.zip
unzip ministra-5.6.1.zip
rm -rf *.zip

# Create Database ( there is some changes due to Mysql 8.X)
mysql -uroot -e "CREATE DATABASE stalker_db;"
mysql -uroot -e "CREATE USER stalker@localhost IDENTIFIED BY '1';"
mysql -uroot -e "GRANT ALL PRIVILEGES ON stalker_db.* TO stalker@localhost WITH GRANT OPTION;"

# Install NPM  2.5.11
sudo apt-get -y -u install npm
sudo npm install -g npm@2.15.11
sudo ln -s /usr/bin/nodejs /usr/bin/node

# Set the Server Timezone to EDT
echo "America/Toronto" > /etc/timezone
dpkg-reconfigure -f noninteractive tzdata

# MySQL Settings
echo 'sql_mode=""' >> /etc/mysql/mysql.conf.d/mysqld.cnf
echo 'default_authentication_plugin=mysql_native_password' >> /etc/mysql/mysql.conf.d/mysqld.cnf   # For MySQL 8.X

/etc/init.d/mysql restart

# PHP Settings
echo "short_open_tag = On" >> /etc/php/7.0/apache2/php.ini

# Ministra custom.ini
# Edit configuration using test editor

vim /var/www/html/stalker_portal/server/costom.ini 
cat /var/www/html/stalker_portal/server/costom.ini # Check file content after configuration
default_stb_status = 0

auto_add_stb = false

; 1 - on, 0 - off
;default_stb_status = 0

auth_url = http://localhost/stalker_portal/server/tools/auth_simple.php

enable_service_button = true

display_menu_after_loading = true

enable_m3u_file = false


# Apache config 
# Edit configuration using test editor

vim /etc/apache2/sites-enabled/000-default.conf
cat /etc/apache2/sites-enabled/000-default.conf # Check file content after configuration
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        <Directory /var/www/html/stalker_portal/>
                Options -Indexes -MultiViews
                AllowOverride ALL
                Require all granted
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

/etc/init.d/apache2 restart

# Pear
pear channel-discover pear.phing.info
pear install --alldeps phing/phing-2.15.2   # Phing V2.16.4 has an issue with PHP7.0 and PHP7.1

# Fix Smart Launcher Applications
mkdir /var/www/html/.npm
chmod 777 /var/www/html/.npm

# Phing :)
cd /var/www/html/stalker_portal/deploy
phing

# Remove system I/O limiter for mysql-server
cp /lib/systemd/system/mysql.service /etc/systemd/system/
echo "LimitNOFILE=infinity" >> /etc/systemd/system/mysql.service
echo "LimitMEMLOCK=infinity" >> /etc/systemd/system/mysql.service
systemctl daemon-reload
systemctl restart mysql
# By Default, In Ubuntu 18.04 mysql-server's root user can access it without a password.
```

Now you can access Ministra TV platform's admin panel at `http://[Your server's IP address]/stalker_portal/` 
