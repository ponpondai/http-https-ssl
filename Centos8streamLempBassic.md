# 在CENTOS8 STREAM建立LEMP基本架構
參考
https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-8

## 1.安裝nginx
```
#centos8開始預設使用dnf作為管理套件
$ sudo dnf update
$ sudo dnf install nginx
#啟動nginx
$ sudo systemctl start nginx
$ sudo systemctl enbale nginx
#防火牆允許nginx
$ sudo firewall-cmd --permanent --add-service=http
#驗證防火牆
$ sudo firewall-cmd --permanent --list-all
```
![](https://i.imgur.com/3dxcCOM.png)
```
#重新加仔防火牆
$ sudo firewall-cmd --reload
#測試是否正常運行
#請連限制對外ip看能否連到nginx網站
```
![](https://i.imgur.com/WaYenQZ.png)

## nginx安裝完成
d(`･∀･)bd(`･∀･)bd(`･∀･)b



# 2.安裝mariadb
```
$ sudo dnf install mariadb-server
#啟動mariadb
$ sudo systemctl start mariadb
$ sudo systemctl enable mariadb
#安全設定
$ sudo mysql_secure_installation
各項設定，詳情確認網站
#測試是否能進入
$ sudo mysql
```
# 3.安裝PHP-FPM PHP-MYSQLND
```
## 安裝php7.4
#由於centos8 stream 沒有直接提供7.4版本，所以我們手動拉入更新清單

$ sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo yum -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
$ sudo dnf -y install dnf-utils
$ sudo dnf module install php:remi-7.4
$ sudo dnf update
$ sudo yum list php

#安裝需要php套件
$ sudo yum install php-fpm php-mysqlnd
$ sudo setsebool -P httpd_execmem 1
#啟動php-fpm&重啟httpd
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
$ sudo systemctl restart nginx

#產生網頁確認php是否成功
$ sudo nano /usr/share/nginx/html/info.php
```
```
<?php

phpinfo();
```
```
#進入ip看有沒有成功
http://server_host_or_IP/info.php
```
![](https://i.imgur.com/MkRuCGt.png)


# LEMP基本架構成功啦!!
ヽ(✿ﾟ▽ﾟ)ノヽ(✿ﾟ▽ﾟ)ノヽ(✿ﾟ▽ﾟ)ノヽ(✿ﾟ▽ﾟ)ノヽ(✿ﾟ▽ﾟ)ノ