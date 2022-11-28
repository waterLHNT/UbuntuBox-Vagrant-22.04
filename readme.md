# Cài đặt UbuntuBox 22.04
<!-- TOC -->

- [Cài đặt UbuntuBox 22.04](#ubuntu)
  - [1. Chuẩn bị file vagrant box](#1-chuẩn-bị-file-vagrant-box)
  - [2. Cài đặt LAMP](#2-cài-đặt-lamp)
    - [2.1. Điều kiện](#21-điều-kiện)
    - [2.2 PHP 8.1](#22-php-81)
    - [2.3 Apache](#23-apache)
      - [Cài đặt và kiểm tra apache](#cài-đặt-và-kiểm-tra-apache)
    - [2.4 MySQL 8.0.30](#24-mysql-8030)
      - [Cài đặt MySQL](#cài-đặt-mysql)
      - [Bảo mật MySQL Server](#bảo-mật-mysql-server)
    
  - [3. Cài đặt Composer 2.4.2](#3-cài-đặt-composer-242)
  - [4. Cài đặt NodeJS 18 và npm](#4-cài-đặt-nodejs-18-và-npm)
  - [5. Cài đặt Git và Ungit(Options)](#5-cài-đặt-git-và-ungit(options))
    - [Cài đặt git](#cài-đặt-git)
    - [Cài đặt Ungit(Options)](#cài-đặt-ungit(options))
  - [6. Rainloop + Dovecot + Postfix](#6-rainloop--dovecot--postfix)
    - [6.1. Cài đặt Postfix](#61-cài-đặt-postfix)
    - [6.2. Cài đặt Dovecot](#62-cài-đặt-dovecot)
    - [6.3. Cài đặt Rainloop](#63-Cài-đặt-rainloop)
  - [7. Adminer 4.8.1(Options)](#7-adminer-481(options))
  - [8. phpMyAdmin](#8-phpmyadmin)
  - [9. Samba 4 dùng để share file](#9-samba-4-dùng-để-share-folder)
  - [10. Cài đặt SSL cho domain](#10-cài-đặt-ssl-cho-domain)

<!-- /TOC -->

## 1. Chuẩn bị file vagrant box
Lưu ý: Để có thể làm theo hướng dẫn cần phải cài đặt `vagrant` và `virtual box`.  
Link tải `(2 phần mềm này để ở mặc định ổ C không đổi.)`:  
Vagrant (https://www.vagrantup.com/downloads).  
Virtual (https://www.virtualbox.org/).  
  
Sau khi cài xong 2 cái trên bắt đầu cài đặt box.  
Vào thư mục vagrant, tạo 1 thư mục con chứa project box.  
VD: `..\Vagrant\project\Ubuntu_22-04`  
Mở cmd gõ các lệnh sau `(Quyền admin)`
```cmd
$ vagrant init boxen/ubuntu-22.04-x86_64
$ truncate Vagrantfile -s 0
```

Copy nội dung file `Vagrantfile` có `trên git` vào file vừa tạo.  
Có thể tìm hiểu thêm về các thông số `config file Vagrantfile` (https://developer.hashicorp.com/vagrant/docs/vagrantfile).  
Lưu ý các mục quan trọng thay đổi để `access ssh`:
```Ruby
config.vm.box = "boxen/ubuntu-22.04-x86_64"
config.vm.network "public_network", ip: "Your IP", netmask: "255.255.240.0", gateway: "10.11.0.1"
```

Sau khi copy nội dung file xong, quay lại cmd gõ lệnh:
```cmd
$ vagrant up --provision
```

Chờ download box và cài đặt box (https://app.vagrantup.com/boxen/boxes/ubuntu-22.04-x86_64) `(Cài đặt hệ điều hành Ubuntu)`.  
Lưu ý: Trong quá trình cài đặt nếu hiện `config interface` thì bấm 1 => enter, nếu không có thì bỏ qua:
```cmd
default: Which interface should the network bridge to? 1
```

Lệnh ngừng, để kiểm tra box cài được chưa thì mở phần mềm `Virtual box` lên, nếu status hiện `Running` là thành công.  
Tham khảo 1 số lệnh cơ bản của vagrant (https://www.drupal.org/node/2008794).  
Sau khi đã cài xong, quay lại cmd `ssh` vào vagrant box:
```cmd
$ vagrant ssh
```

Cài `password` login quyền root:
```cmd
# sudo passwd
```

Login với quyền root:
```cmd
# su
```

Sửa `ssh config` (Quan trọng để máy khác có thể ssh vào)
```cmd
# vim /etc/ssh/sshd_config
```

```Ruby
Tìm và thay đổi thành `PasswordAuthentication yes` bỏ #. 
Tìm và thay đổi `PermitRootLogin yes` bỏ #.
```

Lưu file và khởi động lại ssh:
```cmd
# service ssh restart
# exit
```

Cài `net-tools`
```cmd
# apt install net-tools
```

## 2. Cài đặt LAMP
### 2.1. Điều kiện
Cập nhật repositories:
```cmd
# apt update
```

Thêm kho lưu trữ `“ondrej/php”`:
```cmd
# add-apt-repository ppa:ondrej/php
```

### 2.2 PHP 8.1
Cài đặt `php`:
```cmd
# apt install php8.1
# php -v
Output
PHP 8.1.10 (cli) (built: Sep 18 2022 10:26:02) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.10, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.10, Copyright (c), by Zend Technologies
```

Cài `extensions php 8.1`:
```cmd
# apt install php8.1-{bcmath,xml,fpm,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi}
```

Kiểm tra các `extentions` đã cài:
```cmd
# php -m
```

`(Options)` Lưu ý: Muốn cài php version khác cũng thực hiện như trên, thay đổi `8.1 => ver php muốn cài`.

### 2.3 Apache
#### Cài đặt và kiểm tra apache
```cmd
# apt install apache2 libapache2-mod-php8.1 -y
```

`(Options)` Lưu ý: Nếu có cài ver php khác thì chạy lệnh trên 1 lần, thay đổi `8.1 => ver php đã cài`.  
Mở port `80` và `43` bằng UFW firewall:
```cmd
# ufw allow 80/tcp
# ufw allow 43/tcp
```

### 2.4 MySQL 8.0.30
#### Cài đặt MySQL
```cmd
# apt-get install mysql-server
# mysql -v
Output
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.30-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Reading history-file /root/.mysql_history
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by 'Cba@123456';
mysql> exit;
```
#### Bảo mật MySQL Server
```cmd
# mysql_secure_installation
Nhập pass: Cba@123456 cho user root để chạy lệnh tiếp theo
Lưu ý mấy chỗ : Y hoặc : N, nhập theo ouput ở dưới.
Output

Securing the MySQL server deployment.

Enter password for user root:

The existing password for the user account root has expired. Please set a new password.

New password:

Re-enter new password:
The 'validate_password' component is installed on the server.
The subsequent steps will run with the existing configuration
of the component.
Using existing password for root.

Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : N

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

Vào `check mysql`:
```cmd
mysql -u root -p
Enter password: Your pass mysql
Output
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 23
Server version: 8.0.30-0ubuntu0.22.04.1 (Ubuntu)

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> exit
```

#### Xác minh thiết lập LAMP
```cmd
$ echo '<?php phpinfo(); ?>' > /var/www/html/info.php
```
Kiểm tra thông tin php: `http://your-ip-address/info.php/`
## 3. Cài đặt Composer 2.4.2
```cmd
# apt update
# apt install php-cli unzip
# curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php
# HASH=`curl -sS https://composer.github.io/installer.sig`
# php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Output
Installer verified
# php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer
# composer -v
Output
Continue as root/super user [yes]? y
   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.4.2...
...
```

## 4. Cài đặt NodeJS 18 và npm
```cmd
# apt update && sudo apt upgrade
# curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
# apt-get install nodejs
```

Kiểm tra version:
```cmd
# node -v
# npm -v
```

## 5. Cài đặt Git và Ungit(Options)
### Cài đặt git
```cmd
# apt install git
# git --version
```

### Cài đặt Ungit(Options)
```cmd
# sudo -H npm install -g ungit
```

Tạo file `.ungitrc` ở thư mục bắt đầu vào:
```cmd
# touch .ungitrc
# sudo nano .ungitrc
```

Thêm nội dung bên dưới vào file `.ungitrc`:
```Ruby
{
  "port": 8448,
  "bugtracking": true,
  "logRESTRequests": false,
  "ungitBindIp": "0.0.0.0",
  "autoFetch": false,
  "sendUsageStatistics": true,
  "socketId": 3
}
```

Chạy `ungit service`:
```cmd
# ungit
http://IP:8448/#/repository?path=/root
```

## 6. Rainloop + Dovecot + Postfix
### 6.1. Cài đặt Postfix
```cmd
# apt update
# apt -y install postfix sasl2-bin
# apt-get install postfix-pcre
```

Chỉnh sửa file:  
Truy cập link sau để xem chi tiết (https://www.server-world.info/en/note?os=Ubuntu_22.04&p=mail&f=1).

### 6.2. Cài đặt Dovecot
```cmd
# apt -y install dovecot-core dovecot-pop3d dovecot-imapd
```

Chỉnh sửa file:  
Truy cập link sau để xem chi tiết (https://www.server-world.info/en/note?os=Ubuntu_22.04&p=mail&f=2).
### 6.3. Cài đặt Rainloop
```cmd
# curl -o rainloop-latest.zip https://www.rainloop.net/repository/webmail/rainloop-latest.zip
# mkdir -p /var/www/vhosts/rainloop
# unzip rainloop-latest.zip -d /var/www/vhosts/rainloop/
```

Config thư mục rainloop:
```cmd
# find /var/www/vhosts/rainloop -type d -exec chmod 755 {} \;
# find /var/www/vhosts/rainloop -type f -exec chmod 644 {} \;
# chown -R www-data:www-data /var/www/vhosts/rainloop/
```

Config với `apache`:
```cmd
# vim /etc/apache2/sites-available/rainloop.conf
```

Thêm nội dung sau vào file `rainloop.conf`:  
Lưu ý: chỉnh sửa `ServerName` theo domain riêng được cấp.
```Ruby
<VirtualHost *:80>
  ServerName rainloop.terra.vm
  DocumentRoot "/var/www/vhosts/rainloop/"
  ErrorLog "/var/log/apache2/rainloop_error_log"
  TransferLog "/var/log/apache2/rainloop_access_log"
  <Directory "/var/www/vhosts/rainloop">
      Options Indexes FollowSymLinks MultiViews
      AllowOverride all
      Require all granted
  </Directory>

</VirtualHost>
```

```cmd
# a2ensite rainloop.conf
# systemctl reload apache2
```

Cấu hình `Rainloop`:  
Truy cập vào domain rainloop `rainloop.terra.vm/rainloop/?admin` và đăng nhập với thông tin `admin|12345`.  
Cấu hình `Domains`:  
![rainloop](/img/2.PNG)
  
Bấm `button test` để kiểm tra port:  
![rainloop](/img/admin_rainloop.PNG)
  
Cấu hình `Login`:  
![rainloop](/img/login_rainloop.PNG)
  
Cài đặt `mailx`:
```cmd
# sudo apt-get update
# sudo apt-get -y install bsd-mailx
# sudo apt -y install mailutils
```

Tạo user `catchall`:
```cmd
# adduser catchall
# passwd catchall
# vim /etc/aliases.regexp
Add below contents
/(?!^root$|^catchall$)^.*$/ catchall
```

Chỉnh file `main.cf`:
```cmd
# cd /etc/postfix/
# cp main.cf main.cf.bak
# vim main.cf
Chỉnh sửa file như sau:
# uncomment
home_mailbox = Maildir/

# change this line
alias_maps = hash:/etc/aliases, pcre:/etc/aliases.regexp

# add newline
transport_maps = pcre:/etc/postfix/transport_maps
```

Chỉnh sửa file `aliases`:
```cmd
# vim /etc/aliases
Đổi postmaster: root thành postmaster: catchall

# vim /etc/postfix/transport_maps
# Add below contents
/^.*@.*$/ local

Tạo file rỗng
# vim /etc/postfix/transport
```

Chạy các lệnh sau:
```cmd
postalias /etc/aliases
postmap /etc/postfix/transport
systemctl restart postfix
systemctl restart dovecot
systemctl enable dovecot
```

```cmd
mail -s "This is Subject" -r "sender<sender@mail.com>" someone@example.com
```
Vào `url rainloop` kiểm tra mail.

## 7. Adminer 4.8.1(Options)
```cmd
# apt update 
# apt upgrade
# apt install adminer
# a2enconf adminer
# systemctl reload apache2
```

https://your-ip-address/adminer/
## 8. phpMyAdmin
```cmd
# apt install phpmyadmin php-mbstring php-zip php-gd php-json php-curl
```

Phần select lưu ý bấm thêm `space để có dấu * chọn apache2` => bấm enter.  
Lưu ý Chọn `abort` để hủy không config vì đã setup trên `LAMP`.  
Tiếp tục gõ theo lệnh dưới:
```cmd
# mysql -u root -p
> UNINSTALL COMPONENT "file://component_validate_password";
> exit;
# apt install phpmyadmin
# mysql -u root -p
> INSTALL COMPONENT "file://component_validate_password";
> exit;
# phpenmod mbstring
# systemctl restart apache2
```

Cài đặt cấu hình `phpMyadmin`:
```cmd
# mysql -u root -p
Enter password:
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY 'Your password with policy | MEDIUM';
mysql> SELECT user,authentication_string,plugin,host FROM mysql.user;
mysql> exit;
# vim /etc/phpmyadmin/config-db.php
Change to $dbuser='root';
Change to $dbpass='Your password with policy | MEDIUM';
```

## 9. Samba 4 dùng để share folder
```cmd
# apt install samba
```

Khởi động và bật `smbd`, `nmbd`:
```cmd
# systemctl enable smbd
# systemctl enable nmbd
# systemctl start smbd
# systemctl start nmbd
```

Cài đặt cấu hình chia sẻ mà người khác có thể truy cập vào thư mục:  
Chỉnh sửa file `/etc/samba/smb.conf`:
```cmd
# vim /etc/samba/smb.conf
[global]
        workgroup = WORKGROUP
        #Add line
        map to guest = bad user
        force user = root
#Add end line
[vhosts]
        path = /var/www/vhosts
        browsable = yes
        writable = yes
        guest ok = yes
        read only = no
        follow symlinks = yes
```

Khởi động lại `smbd`:
```cmd
# systemctl restart smbd
# systemctl restart nmbd
```

Kiểm tra `ngoài thư mục ssh`.
## 10. Cài đặt SSL cho domain
Bật `mode ssl` sau đó khởi động lại `apache2`:
```cmd
# a2enmod ssl
# service apache2 restart
```

Tạo `key ssl`:
```cmd
# sudo mkdir /etc/apache2/ssl
# sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt
```

Lưu ý -days 365 có thể đăng ký nhiều hơn  
Sau khi tạo key thì sẽ hiện những thông tin cần nhập, chỉ cần chú ý `"Common Name", nhập your-ip-address chỗ này`.  
Tạo file mặc định `config SSL cho your-ip-address`:
```cmd
# sudo nano /etc/apache2/sites-available/default.conf
```

Thêm nội dung sau vào file:
```Ruby
<VirtualHost *:443>
        ServerAdmin webmaster@localhost
        ServerName your-ip-address:443

        SSLEngine on
        SSLCertificateFile /etc/apache2/ssl/apache.crt
        SSLCertificateKeyFile /etc/apache2/ssl/apache.key
</VirtualHost>
```

Lưu file và khởi động lại apache2:
```cmd
# sudo a2ensite default.conf
Những file config 000-default, *.conf sửa *:80 thành *:443, thêm 3 dòng SSL như trên.

# sudo service apache2 restart
```

Check https `(https://your-ip-address/info.php/)`.