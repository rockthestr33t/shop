# Install annularis on ubuntu16.04 LTS
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y 

# install dependencies ubuntu16.04
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 4F4EA0AAE5267A6C
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update

sudo apt-get install -y nginx mysql-server mysql-client libapache2-mod-php7.1 php7.1 php7.1-mcrypt php7.1-mbstring php-gnupg php7.1-mysql php7.1-gmp php7.1-curl php7.1-bcmath php7.1-gd php7.1-fpm git curl git mcrypt curl unzip atool subversion  



sudo apt-get install -y apache2 mysql-server mysql-client libapache2-mod-php7.0 php7.0 php7.0-mcrypt php7.0-mbstring php-gnupg php7.0-mysql php7.0-gmp php7.0-curl php7.0-bcmath php7.0-gd php7.0-fpm git curl git mcrypt curl unzip atool subversion  

# install dependencies ubuntu14.04
sudo apt-get install apache2 mysql-server mysql-client php5 libapache2-mod-php5 php5-mcrypt php5-gnupg php5-mysql php5-gmp php5-curl php5-gd git

# change installation directory
cd /var/www/

# clone annularis
sudo git clone https://github.com/rockthestr33t/shop.git

# change owneship
sudo chown www-data:www-data -Rv shop

# change dictory
cd shop

# enable rewrite
sudo a2enmod rewrite

# restart apache
sudo service apache2 restart

# install bitcoind
cd /tmp
sudo wget https://bitcoin.org/bin/bitcoin-core-0.15.2/bitcoin-0.15.2-x86_64-linux-gnu.tar.gz
sudo aunpack bitcoin-0.15.2-x86_64-linux-gnu.tar.gz 
cd bitcoin-0.15.2
sudo cp bin/* /usr/local/bin/
sudo bitcoind -daemon -testnet -rpcport="7530" -rpcuser="bitcoinuser" -rpcpassword="bitcoinpass"

# config apache
sudo sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride all/' /etc/apache2/apache2.conf
sudo grep -q "shop" /etc/apache2/sites-enabled/000-default.conf
sudo service apache2 restart

# mysql config
sudo mysql -e "CREATE DATABASE annularis;"
sudo mysql -e "CREATE USER annularis IDENTIFIED BY 'password';"
sudo mysql -e "GRANT ALL PRIVILEGES ON annularis.* TO annularis@localhost IDENTIFIED BY 'password';"

# setup for codeignitter
sudo vim /var/www/shop/install/config/database.php # annularis annularis localhost password
sudo vim /var/www/shop/install/config/bitcoin.php # bitcoinuser bitcoinpass 127.0.0.1 7530
sudo service apache2 restart

# codeignitter instructions
sudo touch /var/www/shop/application/config/database.php && sudo chmod 777 /var/www/shop/application/config/database.php
sudo touch /var/www/shop/application/config/bitcoin.php && sudo chmod 777 /var/www/shop/application/config/bitcoin.php
sudo touch /var/www/shop/application/config/config.php && sudo chmod 777 /var/www/shop/application/config/config.php

# config bitcoin.php
sudo vim /var/www/shop/install/config/bitcoin.php
#blocknotify=/usr/bin/php /var/www/shop/index.php callback block %s 
#alertnotify=/usr/bin/php /var/www/shop/index.php callback alert
crontab -e # and insert
#*/1 * * * * /usr/bin/php /var/www/shop/index.php callback autorun
#*/1 * * * * /usr/bin/php /var/www/shop/index.php callback process

# update database
php /var/www/shop/index.php db

# set critical files to read only 
sudo chmod 755 /var/www/shop -R && sudo chmod 777 /var/www/shop/application/storage/ 

# installed mysql8
wget https://dev.mysql.com/get/mysql-apt-config_0.8.9-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.9-1_all.deb

sudo apt-get update
sudo apt-get install mysql-server # and kep the old config

# install nginx
sudo service apache2 stop
sudo apt-get install nginx

# add group / user
sudo groupadd annularis
sudo useradd -g annularis annularis

# php7.0-fpm conf
sudo cp /etc/php/7.0/fpm/pool.d/www.conf /etc/php/7.0/fpm/pool.d/annularis.conf
sudo vim /etc/php/7.0/fpm/pool.d/annularis.conf
echo " ## edit as following "
echo " # [annularis] "
echo " # user = annularis "
echo " # group = annularis "
echo " # listen = /var/run/php/php7.0-fpm-annularis.sock "
echo " # listen.owner = www-data "
echo " # listen.group = www-data "
echo " # php_admin_ value[disable_functions] = exec,passthru,shell_exec,system "
echo " # php_admin_flag[allow_url_fopen] = off "
sudo vim /etc/php/7.0/fpm/conf.d/10-opcache.ini
## edit as following
# opcache.enable=0

# restart fpm
sudo service php7.0-fpm restart # if problems " sudo apt-get purge php.* "

# install tor
https://www.torproject.org/docs/debian.html.en

sudo apt install apt-transport-https
sudo vim /etc/apt/sources.list.d/
deb https://deb.torproject.org/torproject.org xenial main
deb-src https://deb.torproject.org/torproject.org xenial main

sudo curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
sudo gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -

sudo apt update
sudo apt install tor deb.torproject.org-keyring

# tor config
sudo vim /etc/tor/torrc


# after install 
sudo chmod 755 /var/www/shop -R && sudo chmod 777 /var/www/shop/application/storage/ 

































