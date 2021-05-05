# 設定virtual host
## 用途
假設我們ip有限，但我們卻想要讓多個網站都使用同一個ip那麼我們就可以使用virtual host
___
# 設定檔
在/etc/httpd/conf.d/ 底下的檔案會自行套用進httpd
所以我們產生一個/etc/httpd/conf.d/virtual.conf 檔案在底下放置我們的virtual host設定


$ sudo nano /etc/httpd/conf.d/virtual.conf
```
#第一個網站
<VirtualHost *:80>
    #代表server返回給client的錯誤訊息中包含管理者的信箱(也可以不設定這行)
    ServerAdmin webmaster@www.n.com
    #網站根目錄
    DocumentRoot /www/docs/n
    #網址
    ServerName www.n.com
    #ErrorLog的存放位置
    ErrorLog logs/www.n.com-error_log
    #用戶log的存放位置
    #common代表顯示格式(http://n.sfs.tw/content/index/10147)
    #可以在 /etc/httpd/conf/httpd.conf 中編輯格式
    CustomLog logs/www.n.com-access_log common
</VirtualHost>
#第二個網站依此類推，當然還可以設定443port讓網站使用https
<VirtualHost *:80>
    ServerAdmin webmaster@www.n.com
    DocumentRoot /www/docs/n
    ServerName www.n.com
    ErrorLog logs/www.n.com-error_log
    CustomLog logs/www.n.com-access_log common
</VirtualHost>
```


