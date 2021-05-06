# 如何讓在Centos8 Nginx啟用https認證

## 自行產生簽證
參照
https://blog.gtwang.org/linux/nginx-create-and-install-ssl-certificate-on-ubuntu-linux/



## 需要條件
1.有sudo權限帳號
2.已安裝好Nginx(如沒有請參考centos8LEMP章節#名字可能有些許不同)
3.需要有一組domain#這邊有組測試環境的網域好便宜(https://at.twnic.tw/pages/description.php)
4.此domain以經dns指向你的ip
```
#產生放置金鑰目錄
$ sudo mkdir /etc/nginx/ssl
#產生金鑰
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt
##
內容說明

req：使用 X.509 Certificate Signing Request（CSR） Management 產生憑證。
-x509：建立自行簽署的憑證。
-nodes：不要使用密碼保護，因為這個憑證是 NGINX 伺服器要使用的，如果設定密碼的話，會讓伺服器每次在啟動時書需要輸入密碼。
-days 365：設定憑證的使用期限，單位是天，如果不想時常重新產生憑證，可以設長一點。
-newkey rsa:2048：同時產生新的 RSA 2048 位元的金鑰。
-keyout：設定金鑰儲存的位置。
-out：設定憑證儲存的位置。
##
```
![](https://i.imgur.com/XqS8DjI.png)

```
#修改nginx設定
$ sudo nano /etc/nginx/nginx.conf
如下圖
# 將https打開
# 取消註解
# 加入使用認證金鑰位置
```
![](https://i.imgur.com/ldlB0Hl.png)

```
#重啟伺服器
$ sudo service nginx restart
#連線至網站ip(記得加上https://)
```
連線成功即自簽成功!!!!
恭喜
# ｡:.ﾟヽ(*´∀`)ﾉﾟ.:｡


# 使用三方簽證(不會有安全問題顯示)

參考文件
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-8


## 1.安裝Cerbot套件

```
#安裝需要套件
$ sudo dnf install epel-release
$ sudo dnf install certbot python3-certbot-nginx
```

## 2. 更新防火牆
```
#確認是否有在防火牆可用清單內
$ sudo firewall-cmd --permanent --list-all
```
![](https://i.imgur.com/hN2pofN.png)
##如果沒有執行
```
$ sudo firewall-cmd --permanent --add-service=http

```
##允許HTTPS流量
```
$ sudo firewall-cmd --permanent --add-service=https
#重載防火牆
$ sudo firewall-cmd --reload

```

## 3.取得Certificate

```
#有更多domain可以再多加個-d來增加其他subdomain
$ sudo certbot --nginx -d your_domain

```
申請成功會出現這些訊息
![](https://i.imgur.com/bMj4qdh.png)

連線至https:你的網域名稱確認
![](https://i.imgur.com/YFbn448.png)
左上角沒出現不安全憑證即表示設定成功
# (*´ω`)人(´ω`*)(*´ω`)人(´ω`*)




## 4.自動續訂憑證
由於此憑證有效期限為90天所以等等我們使用命令讓他在60天時進行憑證更新

```
#更新憑證明令
$ sudo certbot renew --dry-run
```
![](https://i.imgur.com/d4MoxLS.png)

使用crontab來進行自動憑證更新
```
#每天續訂兩次
$ sudo crontab -e
```
```
0 0,12 * * * python -c 'import random; import time; time.sleep(random.random() * 3600)' && certbot renew --quiet
```

