# https

## http  80端口

## https 443端口

KPI	公钥基础设施

密钥：  对称密钥		（两边一样）

​				非对称密钥		（公钥和私钥）

https	（http+ssl）

## 生成一对非对称密钥

openssl genrsa  -out /file.key 2048

 genrsa	生成一对非对称密钥

  -out 	将生成的密钥文件输出到指定的路径上

 2048	指定生成密钥的长度	密钥长度越长 越安全

## #生成证书申请的文件

openssl  req -new -key /file.key -out /file.csr -days 365

![](D:\github\jichufuwu\image\Untitled\11.gif)

 -new  新证书的申请

-key 	证书使用的密钥

 -out	指定申请文件存放的位置

  365	证书申请的有效期

## #颁发证书

 openssl x509 -req -in /file.csr -signkey /file.key -out /file.crt -days 365

  x509 颁发证书遵循x509标准

 -in 	申请证书的名称

 -signkey 	使用的密钥文件

 -out  证书存放的位置

-days 	有效天数

cp  /file.*  /usr/local/httpd/conf/

##  vim /usr/local/httpd/conf/extra/httpd-ssl.conf 

144 SSLCertificateFile "/usr/local/httpd/conf/file.crt"

154 SSLCertificateKeyFile "/usr/local/httpd/conf/file.key"

165 SSLCertificateChainFile "/usr/local/httpd/conf/file.csr"

## vim /usr/local/httpd/conf/httpd.conf 

504 Include conf/extra/httpd-ssl.conf

135 LoadModule ssl_module modules/mod_ssl.so

90  LoadModule socache_shmcb_module modules/mod_socache_shmcb.so

systemctl restart httpd

firefox https://127.0.0.1 & 

点高级   添加例外   确认添加例外

![](D:\github\jichufuwu\image\Untitled\呃呃呃.gif)

