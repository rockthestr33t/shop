# Install annularis on ubuntu16.04 LTS
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y 

# install dependencies ubuntu16.04
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 4F4EA0AAE5267A6C
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update

sudo apt-get install -y nginx mysql-server mysql-client libapache2-mod-php7.1 php7.1 php7.1-mcrypt php7.1-mbstring php-gnupg php7.1-mysql php7.1-gmp php7.1-curl php7.1-bcmath php7.1-gd php7.1-fpm git curl git mcrypt curl unzip atool subversion  
phpenmod mysqli mcrypt curl gmp gd


sudo apt-get install -y apache2 mysql-server mysql-client libapache2-mod-php7.0 php7.0 php7.0-mcrypt php7.0-mbstring php-gnupg php7.0-mysql php7.0-gmp php7.0-curl php7.0-bcmath php7.0-gd php7.0-fpm git curl git mcrypt curl unzip atool subversion  
sudo phpenmod mysqli mcrypt curl gmp gd

# install dependencies ubuntu14.04
sudo apt-get install apache2 mysql-server mysql-client php5 libapache2-mod-php5 php5-mcrypt php5-gnupg php5-mysql php5-gmp php5-curl php5-gd git

# change installation directory
cd /var/www/

# clone annularis
sudo git clone https://github.com/rockthestr33t/shop.git

# change owneship
sudo chown www-data:www-data -Rv shop
sudo chown www-data:www-data -R . shop # tested worked
sudo chmod 755 -R . shop


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
sudo service php7.0-fpm restart 

# install tor
https://www.torproject.org/docs/debian.html.en

sudo apt install apt-transport-https
sudo vim /etc/apt/sources
deb https://deb.torproject.org/torproject.org xenial main
deb-src https://deb.torproject.org/torproject.org xenial main

sudo curl https://deb.torproject.org/torproject.org/A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89.asc | gpg --import
sudo gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | apt-key add -

sudo apt update
sudo apt install tor deb.torproject.org-keyring

# tor config
sudo vim /etc/tor/torrc
sudo service tor restart

# after install 
sudo chmod 755 /var/www/shop -R && sudo chmod 777 /var/www/shop/application/storage/ 


# thigs in my mind
install sql BitwaspS1W
https://github.com/S1W/BitWasp/blob/master/install/assets/install.sql

USE annularis;

DROP TABLE bw_currencies;

CREATE TABLE IF NOT EXISTS `bw_currencies` (
  `id` int(9) NOT NULL,
  `name` varchar(40) NOT NULL,
  `symbol` varchar(10) NOT NULL,
  `code` varchar(5) NOT NULL,
  `crypto_magic_byte` varchar(2) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO `bw_currencies` (`id`, `name`, `symbol`, `code`, `crypto_magic_byte`) VALUES
(0, 'Bitcoin', 'BTC', 'BTC', '00'),
(1, 'United States Dollar', '&#36;', 'USD', ''),
(2, 'British Pound Sterling', '&pound;', 'GBP', ''),
(3, 'Euro', '&euro;', 'EUR', '');




DROP TABLE bw_exchange_rates;

CREATE TABLE IF NOT EXISTS `bw_exchange_rates` (
  `id` int(9) NOT NULL AUTO_INCREMENT,
  `time` varchar(20) NOT NULL,
  `usd` decimal(10,4) NOT NULL,
  `eur` decimal(10,4) NOT NULL,
  `gbp` decimal(10,4) NOT NULL,
  `btc` int(11) NOT NULL DEFAULT '1',
  `price_index` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

INSERT INTO `bw_exchange_rates` (`id`, `time`, `usd`, `eur`, `gbp`, `btc`, `price_index`) VALUES
(1, '1396517400', 421.6251, 306.3123, 253.2567, 1, 'CoinDesk'),
(2, '1396519320', 422.1184, 306.6707, 253.5530, 1, 'CoinDesk'),
(3, '1396520220', 437.1317, 317.5407, 263.2079, 1, 'CoinDesk');


DROP TABLE bw_categories;

CREATE TABLE IF NOT EXISTS `bw_categories` (
  `id` int(9) NOT NULL AUTO_INCREMENT,
  `description` varchar(100) NOT NULL default '0',
  `hash` varchar(25) NOT NULL,
  `name` varchar(40) NOT NULL,
  `parent_id` int(9) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `hash` (`hash`),
  KEY `hash_2` (`hash`, `parent_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

sudo vim /var/www/shop/templates_c
monero to bitcoin

cd /var/www/shop
sudo chmod -Rv 777 system
sudo chmod -Rv 777 assets
sudo chmod -Rv 777 templates_c
sudo chmod 777 -R . templates_c

sudo service mysql restart
sudo service php7.0-fpm restart
sudo service nginx restart


# comment / delete out the e-mail part
sudo vim templates_c/5a3fc9d50215658b93470a0fea00c3281eff93a9_0.file.edit.tpl.php
line 93.1<!--
                    <div class="form-group">
                        <div class="col-12">
                            <label class="control-label col-4" for="email_updates">Forward messages to email?</label>
                            <div class="col-6">
                                <label class="radio-inline">
                                    <input type='radio' class="radio" name='email_updates' value='0' <?php if ($_smarty_tpl->tpl_vars['user']->value['email_updates'] == '0') {?>checked<?php }?> /> Disabled 
                                </label>

                                <label class="radio-inline">
                                    <input type='radio' class="radio" name='email_updates' value='1' <?php if ($_smarty_tpl->tpl_vars['user']->value['email_updates'] == '1') {?>checked<?php }?> /> Enabled
                                </label>
                            </div>
                        </div>
                    </div>
              -->







1.
sudo vim application/controllers/Admin.php
38. 'monero' => array('panel' => '/bitcoin',


2.
sudo vim install/assets/install.sql
138.  ('site_description', 'open source monero marketplace'),


3.
sudo vim application/controllers/Admin.php
38.  'monero' => array('panel' => '/bitcoin',
289.  $data['title'] = $this->nav['bitcoin']['heading'];


4.
sudo vim application/language/english/form_validation_lang.php
8.  $lang['check_bitcoin_index'] = 'You must select a valid Bitcoin Index source';
28.   $lang['check_registration_token_charge'] = 'Enter a bitcoin amount, or zero, or default for this amount.';


5.
sudo vim application/libraries/Autorun/Price_Index.php
23/24.   public $config = array('name' => 'Bitcoin Price Index',
        'description' => 'An autorun job to update the Bitcoin exchange rates.',
33.    * Load's the CodeIgniter framework and the Bitcoin library.  


6.
sudo vim application/libraries/Bw_bitcoin.php
337.     $this->CI->logs_model->add('Price Index', 'Unable to fetch exchange rates', 'An attempt to update the Monero 


7.
sudo vim application/models/Order_model.php
414.   : 'You have accepted this order! Visit the orders page to see the monero address!';


8.
sudo vim application/models/Users_model.php
291.   and $info['monero_address'] must be supplied.


9.
sudo vim application/views/smarty_templates/accounts/me.tpl
65.   <div class="col-md-4"><strong>Monero Public Keys</strong></div>


10.
sudo vim application/views/smarty_templates/admin/fees.tpl
108.   <span class="input-group-addon"><i>XMR</i></span>


11.
sudo vim  application/views/smarty_templates/admin/maintenance.tpl
10.   <p>If maintenance mode has been triggered by the bitcoin daemon, or due to an alert reported on github


12.
sudo vim application/views/smarty_templates/bip32/created_addresses.tpl
2.    <p align="justify">Each time a new order is created, a new monero address will show up here, a


13.
sudo vim application/views/smarty_templates/bip32/no_key.tpl
52.   <p align="justify">Using an external wallet is only recommended for advanced users, b


14. 
bitcoin.index
sudo vim 6fe626a2887afcfb6944580388424f7ee234f389_0.file.bitcoin.tpl.php


15.
bitcoin.index footer
sudo vim 9a387c118444f3aa07e361f0d439b0b31992d911_0.file.footer.tpl.php



sudo apt-get install gnupg

gpg —gen-key
gpg —armor —export nishijorna85@gmail.com > public_key.asc
  or
gpg —list-keys
gpg —send-keys PRIMARY_ID


!!!!!!!!!!!!not tested !!!!!!!!!!!!!!!!!!!!!!!!!

Drop TABLE bw_users;

CREATE TABLE IF NOT EXISTS `bw_users` (
  `id` int(9) NOT NULL AUTO_INCREMENT,
  `banned` enum('0','1') DEFAULT '0',
  `block_non_pgp` enum('0','1') DEFAULT '0',
  `entry_paid` enum('0','1') DEFAULT '0',
  `force_pgp_messages` enum('0','1') DEFAULT '0',
  `location` int(3) NOT NULL,
  `login_time` int(20) NOT NULL, DEFAULT '0',
  `display_login_time` enum('0','1') DEFAULT '0',
  `password` varchar(128) NOT NULL,
  `public_key` blob NOT NULL,
  `private_key` blob NOT NULL,
  `private_key_salt` varchar(64) NOT NULL,
  `register_time` int(20) NOT NULL,
  `salt` varchar(128) NOT NULL,
  `wallet_salt` varchar(128) NOT NULL,
  `user_hash` varchar(25) NOT NULL,
  `user_name` varchar(40) NOT NULL,
  `user_role` enum('Buyer','Vendor','Admin') NOT NULL,
  `local_currency` int(11) NOT NULL,
  `completed_order_count` int(9)  DEFAULT '0',
  `totp_secret` varchar(25),
  `totp_two_factor` enum('0','1') DEFAULT '0',
  `pgp_two_factor` enum('0','1') DEFAULT '0',
  PRIMARY KEY (`id`),
  UNIQUE KEY `user_hash` (`user_hash`,`user_name`),
  KEY `user_name` (`user_name`,`user_hash`,`banned`,`entry_paid`,`register_time`,`user_role`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;

INSERT INTO `bw_users` (`banned`, `block_non_pgp`, `entry_paid`, `force_pgp_messages`, `location`, `login_time`, `display_login_time`, `password`, `public_key`, `private_key`, `private_key_salt`, `register_time`, `salt`, `pgp_two_factor`, `user_hash`, `user_name`, `user_role`, `local_currency`, `completed_order_count`, `totp_two_factor`, `totp_secret`) VALUES
('0', '0', '1', '0', 1, 0, '0', '%PASSWORD%', '%PUBLIC_KEY%', '%PRIVATE_KEY%', '%PRIVATE_KEY_SALT%', '%REGISTER_TIME%', '%SALT%', '0', '%USER_HASH%', 'admin', 'Admin', 0, 0, '0', '');






















































