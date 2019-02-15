# Install annularis on ubuntu16.04 LTS
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y 

# install dependencies
sudo apt-get install -y apache2 mysql-server mysql-client libapache2-mod-php7.0 php7.0 php7.0-mcrypt php7.0-mbstring php-gnupg php7.0-mysql php7.0-gmp php7.0-curl php7.0-bcmath php7.0-gd php7.0-fpm git curl git mcrypt curl unzip atool subversion  

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
sudo wget https://bitcoin.org/bin/bitcoin-core-0.17.1/bitcoin-0.17.1-x86_64-linux-gnu.tar.gz
sudo aunpack bitcoin-0.17.1-x86_64-linux-gnu.tar.gz
cd bitcoin-0.17.1
sudo cp bin/* /usr/local/bin/
sudo bitcoind -daemon -testnet -rpcport="7530" -rpcuser="bitcoinuser" -rpcpassword="bitcoinpass"

# config apache
sudo sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride all/' /etc/apache2/apache2.conf
sudo service apache2 restart
sudo grep -q "shop" /etc/apache2/sites-enabled/000-default.conf

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
vim /var/www/shop/install/config/bitcoin.php
#blocknotify=/usr/bin/php /var/www/shop/index.php callback block %s 
#alertnotify=/usr/bin/php /var/www/shop/index.php callback alert
crontab -e # and insert
#*/1 * * * * /usr/bin/php /var/www/shop/index.php callback autorun
#*/1 * * * * /usr/bin/php /var/www/shop/index.php callback process

# update database

php /var/www/shop/index.php db
#chmod 755 /var/www/shop -R && chmod 777 /var/www/shop/application/storage/ 














