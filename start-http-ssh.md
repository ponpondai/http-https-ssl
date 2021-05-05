# 如何讓在Centos8 Apache啟用https認證
1.自行產生簽證(連線時會出現不安全憑證訊息)
2.三方產生簽證(Let's Encrypt)

## 自行產生簽證
參照http://n.sfs.tw/content/index/11830

# 請先安裝 mod-ssl
$ sudo yum install -y mod_ssl

一.自簽ROOT CA
#產生私鑰
$ sudo openssl genrsa  -out RootCA.key 2048
#產生根憑證申請檔
$ sudo openssl req -new -key RootCA.key -out RootCA.req
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:TW
State or Province Name (full name) [Berkshire]:Taiwan
Locality Name (eg, city) [Newbury]:Taichung
Organization Name (eg, company) [My Company Ltd]:Somewhere Co.Ltd.
Organizational Unit Name (eg, section) []:Sleep Dep.
Common Name (eg, your name or your server's hostname) []:hostname.example.com
Email Address []:<按ENTER>

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:<按ENTER>
An optional company name []:<按ENTER>
```
#使用申請檔產生憑證檔
$ sudo openssl x509 -req -days 3650 -sha256 -extensions v3_ca -signkey RootCA.key -in RootCA.req -out RootCA.crt
#產生憑證檔RootCA.crt
接下來要拿根憑證來簽發終端憑證，因為沒有中繼憑證，所以直接來作伺服器憑證，也就是終端憑證。

# 二.製作終端憑證(伺服器憑證)
建立伺服器私鑰
$ sudo openssl genrsa -out ServerCert.key 2048
同上，產生憑證申請檔
$ sudo openssl req -new -key ServerCert.key -out ServerCert.req
```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:TW
State or Province Name (full name) [Berkshire]:Taiwan
Locality Name (eg, city) [Newbury]:Taichung
Organization Name (eg, company) [My Company Ltd]:Eat Co.Ltd.
Organizational Unit Name (eg, section) []:Pizza Dep.
Common Name (eg, your name or your server's hostname) []:hostname.example.com
Email Address []:<按ENTER>
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:<按ENTER>
An optional company name []:<按ENTER>
```
產生流水號檔
$ sudo echo 1000 > RootCA.srl
產生憑證檔一樣簽十年
$ sudo openssl x509 -req -days 3650 -sha256 -extensions v3_req -CA RootCA.crt -CAkey RootCA.key -CAserial RootCA.srl -CAcreateserial -in ServerCert.req -out ServerCert.crt

# 三.SELINUX 及搬移
$ sudo chcon -u system_u -t cert_t ServerCert.key ServerCert.crt RootCA.crt
複製到指定的位置
$ sudo cp ServerCert.key /etc/pki/tls/private/

$ sudo cp *.crt /etc/pki/tls/certs

# 四、APACHE上設定

修改 /etc/httpd/conf.d/ssl.conf
```
# 註解這兩項
#SSLCertificateFile /etc/pki/tls/certs/localhost.crt
#SSLCertificateKeyFile /etc/pki/tls/private/localhost.key

# 修改
Listen 443
ServerName hostname.example.com:443
SSLEngine on

# 加入這三項
SSLCertificateFile /etc/pki/tls/certs/ServerCert.crt
SSLCertificateKeyFile /etc/pki/tls/private/ServerCert.key
SSLCACertificateFile /etc/pki/tls/certs/RootCA.crt
```
重新啟動 apache
$ sudo systemctl restart httpd
----------------------------------------
## 檢查
連線至網頁確認使否認證成功


# 使用三方認證(不會有安全問題顯示)
參照
https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-centos-8

# 請先確認virtual host是否已經設定，如無設定請參考virtual host設定章節
$ sudo cat /etc/httpd/conf.d/virtual.conf
會類似如下
```
.
.
.

<VirtualHost *:443>
    ServerName www.example.tw
	ServerAlias example.tw
    ServerAdmin wewe987001@gmail.com
    DocumentRoot /var/www/html/vesc
	
	Include /etc/letsencrypt/options-ssl-apache.conf
	SSLCertificateFile /etc/letsencrypt/live/www.example.tw/cert.pem
	SSLCertificateKeyFile /etc/letsencrypt/live/www.example.tw/privkey.pem
	SSLCertificateChainFile /etc/letsencrypt/live/www.example.tw/chain.pem
</VirtualHost>
.
.
.
```

#新增套件清單
$ sudo dnf install epel-release
#安裝必要套件
$ sudo dnf install certbot python3-certbot-apache mod_ssl


#為自己的domain申請Certificate
#使用certbot向Let’s Encrypt請求ssl Certificate
$ sudo certbot --apache -d example.com

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/example.com/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/example.com/privkey.pem
   Your cert will expire on 2020-09-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

#驗證
連線到你的domain
或是連線到https://www.ssllabs.com/ssltest/analyze.html?d="example.com"
(記得改domain)


# 更新憑證
$ sudo certbot renew --dry-run

# 自動更新憑證
$ echo "0 0,12 * * * root python3 -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew -q" | sudo tee -a /etc/crontab > /dev/null


其他使用方式
參考
https://certbot.eff.org/lets-encrypt/centosrhel8-apache
好處可以使用snap幫忙進行cron job更新憑證
壞出snap core 更新失敗









