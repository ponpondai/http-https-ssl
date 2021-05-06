# 建立LAMP in Centos8 Stream

## 更新&升級管理套件清單

$ sudo yum -y update

## 安裝Apache
```
$ sudo yum install -y httpd
#啟動httpd
$ sudo systemctl start httpd
#開機自動啟動
$ sudo systemctl enable httpd
#允許防火牆使用httpd
$ sudo firewall-cmd --permanent --zone=public --add-service=http
$ sudo firewall-cmd --reload

## 安裝php7.4
#由於centos8 stream 沒有直接提供7.4版本，所以我們手動拉入更新清單
```
$ sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
$ sudo yum -y install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
$ sudo dnf -y install dnf-utils
$ sudo dnf module install php:remi-7.4
$ sudo dnf update
$ sudo yum list php
```
#安裝需要php套件
$ sudo yum install php-fpm php-mysqlnd
$ sudo setsebool -P httpd_execmem 1
#啟動php-fpm&重啟httpd
$ sudo systemctl start php-fpm
$ sudo systemctl enable php-fpm
$ sudo systemctl restart httpd

#產生網頁確認php是否成功
$ sudo vi /var/www/html/info.php
```
<?php
phpinfo();
?>
```
#連線至網頁(ip)如有出現詳細資訊即成功
## 安裝mysql
$ sudo yum install mysql-server
$ sudo systemctl start mysqld.service
$ sudo systemctl enable mysqld.service
#安全性設定
$ sudo mysql_secure_installation
#安全設定可以參照以下網站
http://pejslin.blogspot.com/2017/03/mysqlsecureinstallation.html

## LAMP設定完成(*´ω`)人(´ω`*)
下一篇請參照apache啟用https
ヽ(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)人(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)ﾉ
ヽ(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)人(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)ﾉ
ヽ(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)人(∀ﾟ )人(ﾟ∀ﾟ)人( ﾟ∀)ﾉ