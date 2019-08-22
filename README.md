# Environment Construction Tutorial
* Credit to [Jiawei Li](17818521030@163.com)

To run GSV and mGSV, you need to install **PHP5.3.20 or higher but no more than PHP5.50 + Apache or Nginx + MySQL** and make sure that **PHP has the MySQL extension and GD extension enabled**. Because mGSV needs to use MySQL extensions to operate on MySQL, store collinear data and use the GD library to draw the picture.

If your existing php version is php7 or is not installed PHP, you should build an environment that can use PHP5 because PHP5 and PHP7 have big differences, PHP7 has abandoned some extension libraries. For example, the mysql extension in PHP5 has been deprecated in PHP7 and replaced with the mysqli extension. This will causes a series of mysql extension functions in the GSV and mGSV script code to be unavailable, causing an error.

The following is a tutorial on configuring the environment required to run the GSV and mGSV in **CentOS**.

* **Note: Requires root privileges**

## Section I: Install Nginx

* Install gcc and g++
```bash
yum -y install gcc automake autoconf libtool make
yum install gcc gcc-c++
```
* Install PCRE
```bash
cd /usr/local/src
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
tar -xzvf pcre-8.39.tar.gz
cd pcre-8.39
./configure
make
make install
```
* Install zlib
```bash
cd /usr/local/src
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -xzvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make
make install
```
* Install Nginx
```bash
cd /usr/local/src
wget http://nginx.org/download/nginx-1.1.10.tar.gz
tar -zxvf nginx-1.1.10.tar.gz
cd nginx-1.1.10
./configure
make
make install
```
* Configure Nginx
* Because I installed Apache before and Apache uses port 80. So I configured Nginx to use port 8089.

```bash
vim /usr/local/nginx/conf/nginx.conf
```

![nginxConfig](https://github.com/qunfengdong/GSV/blob/master/img/nginxConfig.png)

* Running Nginx
```bash
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf

```
* Put the web file in /usr/local/nginx/html/ . You can now successfully access the web page in the browser. (i.e., http://YOUR_IP:8089/)
* You will see the following:

![nginxFinish](https://github.com/qunfengdong/GSV/blob/master/img/nginxFinish.png)

## Section II: Install PHP
* Install ibpng and freetype
```bash
conda install -c conda-forge libpng
conda install -c conda-forge freetype
```
* Install GD library
```bash
wget https://github.com/libgd/libgd/releases/download/gd-2.2.4/libgd-2.2.4.tar.gz
```
```bash
tar -xzvf libgd-2.2.4.tar.gz
cd libgd-2.2.4
./configure --prefix=/usr/local/gd2 --with-png --with-freetype
make
make install
```
* Download the required PHP version
```bash
wget http://museum.php.net/php5/php-5.3.20.tar.gz
```
* Compile PHP
```bash
tar -xzvf php-5.3.20.tar.gz
cd php-5.3.20
./configure --enable-fpm --enable-mysqlnd --with-mysql=/opt/lampp/ --without-sqlite --without-pdo-sqlite --with-gd --with-freetype-dir=/usr/include/freetype2/freetype/
make
make install
```
* Obtain and move configuration files to their correct locations
```bash
cp php.ini-development /usr/local/php-5.3.20/php.ini
```
* Load up php.ini
```bash
vim php.ini
```
* Open the mysql and gd extensions. Find and modify the following:

![phpModify_1](https://github.com/qunfengdong/GSV/blob/master/img/phpModify_1.png)

* Locate cgi.fix_pathinfo= and modify it as follows:

![phpModify_2](https://github.com/qunfengdong/GSV/blob/master/img/phpModify_2.png)

## Section III: Configure Nginx and php-fpm so that web pages can use php5 that you just installed

* Obtain and move php-fpm.conf to their correct locations
```bash
cp /usr/local/etc/php-fpm.conf.default /usr/local/sbin/php-fpm.conf

```
* Enter **/usr/local/sbin** and you should see php-fpm and php-fpm.conf
```bash
cd /usr/local/sbin
```
![phpModify_3](https://github.com/qunfengdong/GSV/blob/master/img/phpModify_3.png)
* Modify the php-fpm.conf file

![phpModify_4](https://github.com/qunfengdong/GSV/blob/master/img/phpModify_4.png)

* Running php-fpm
```bash
/usr/local/sbin/php-fpm
```
* Modify the nginx.conf file
```bash
cd /usr/local/nginx/conf
vim nginx.conf
```
![phpModify_5](https://github.com/qunfengdong/GSV/blob/master/img/phpModify_5.png)

* Restart the Nginx
```bash
/usr/local/nginx/sbin/nginx -s reload
```
* Write a PHP script to test if the configuration is successful
```bash
cd /usr/local/nginx/html/
vim phptest.php
```
input:
```
<?php
phpinfo();
?>
```
* Access phptest.php scripts through your browser (i.e., http://YOUR_IP:8089/phptest.php)
* Displays this content indicating that the configuration was successful.
![phpFinal](https://github.com/qunfengdong/GSV/blob/master/img/phpFinal.png)

# mGSV Installation Guide

## System Requirements

The current version of Genome Synteny Viewer was only tested on Ubuntu 10.04 OS. The minimum requirements include: 
* Apache 1.3 or higher (http://www.apache.org)
* PHP 5.3.20 but no more than PHP5.50  (http://www.php.net/)
* MySQL 5.0 or higher (http://dev.mysql.com/)
* GD library. (http://php.net/manual/en/book.image.php)

If you successfully completed the above Environment Construction Tutorial, you have everything installed.

## MySQL Database Setup

* Log into MySQL, replace with your root password
```bash
mysql -u root -p <password>
```
* Create 'mgsv' database
```bash
CREATE DATABASE mgsv;
```
* Create a user 'mgsv'
```bash
CREATE USER 'mgsv'@'localhost' IDENTIFIED BY 'mgsvpass';
```
* Set privileges to 'mgsv' to use database
```bash
GRANT SELECT, INSERT, CREATE, DROP ON mgsv.* TO 'mgsv'@'localhost';
```
* Create table "userinfo" in 'mgsv' database by executing the following MySQL command
```bash
use mgsv;
```
```bash
CREATE TABLE IF NOT EXISTS `userinfo` (
	`id` int(10) NOT NULL AUTO_INCREMENT,
	`email` text NOT NULL,
	`hash` text NOT NULL,
	`synfilename` text NOT NULL,
	`annfilename` text NOT NULL,
	`url` text NOT NULL,
	`session_id` text NOT NULL,
	`annImage`   int(5) NOT NULL,
	`create_on` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
	PRIMARY KEY (`id`)
) ENGINE=MyISAM  DEFAULT CHARSET=latin1 AUTO_INCREMENT=3 ;
```
## Set up mGSV

Download the source code
git clone https://github.com/qunfengdong/mGSV.git
Create a folder named mgsv in the DocumentRoot of Nginx web server. By default, the DocumentRoot of Nginx is /usr/local/nginx/html/
cd /usr/local/nginx/html
mkdir mgsv
chmod 777 mgsv
Enter 'mgsv' directory and create a folder named tmp. Execute a Linux command to change permission for 'tmp' folder. It has to be world accessible to store the uploaded files.
cd mgsv
mkdir tmp
chmod -R 777 tmp
Move all files under the downloaded mGSV folder to the mgsv folder
Under 'mgsv' folder you should see a file called 'Arial.ttf', Copy it to /usr/share/fonts/truetype/, you may need the sudo privilege, talk to your system administrator if necessary.
cp Arial.ttf /usr/share/fonts/truetype/
Modify the setting.php file. File path is /usr/local/nginx/html/mgsv/lib/
cd /usr/local/nginx/html/mgsv/lib/
vim setting.php
Add the following information:

$database_name = 'mgsv';
$database_user = 'mgsv';
$database_pass = 'mgsvpass';
$database_host = 'localhost';
mgsv1

This completes installation, you can now open mgsv/index.php
(i.e., http://<YOUR_SERVER_DOMAIN_NAME>/mgsv/index.php)

The mGSV web server is accessible at http://cas-bioinfo.cas.unt.edu/mgsv and the open-sourced software is also available from the web site for local installation under the terms of the GNU General Public License (http://www.gnu.org/licenses/gpl.html). mGSV is portable across Linux distributions, and compatible with PHP 5.3.3 (or higher version) and MySQL 5.0 (or higher version). mGSV installation has been tested on Debian Lenny and Ubuntu Lynx. mGSV can be viewed with FireFox 3.6.15, Safari 3.0 and Chrome 11.0. The mGSV website will be updated to contain the latest information on operating systems and software compatibility.

Refer to INSTALLATION file for mGSV installation.
