# 生成https自签名证书

域名: ccy652.com

# 生成证书 在命令窗口执行如下命令回车 
`keytool -genkey -v -alias ams -keyalg RSA -keystore ams.keystore -validity 36500`

keytool -genkeypair -alias ams -keypass 123456 -storepass 123456 \
    -dname "C=CN,ST=GD,L=SZ,O=vihoo,OU=dev,CN=ccy652.com" \
    -keyalg RSA -keysize 2048 -validity 36500 -keystore ams.keystore

注意：此处的你的名字与姓氏不能乱写 要写你服务器的ip或者域名  密钥库的密码至少必须6个字符
其中 -genkey 是生成证书   -alias tomcat 是别名 -keyalg RSA 加密方式  tomcat.keystore 是要生成的证书名称  -validity 36500 表示的是有效期 36500天=100年 其他参数说明可以在cmd中输入keytool查看


# 为客户端生成证书 文件为p12类型的证书
`keytool -genkey -v -alias client -keyalg RSA -storetype PKCS12 -keystore client.p12 -validity 36500`

keytool -list -keystore ams.keystore -storepass 123456 

# 导出证书

keytool -exportcert -keystore ams.keystore -file ams.cer -alias ams -storepass 123456

# 自己制作ssl证书：自己签发免费ssl证书，为nginx生成自签名ssl证书

首先执行如下命令生成一个key
openssl genrsa -des3 -out ssl.key 1024

然后他会要求你输入这个key文件的密码。不推荐输入。因为以后要给nginx使用。每次reload nginx配置时候都要你验证这个PAM密码的。
由于生成时候必须输入密码。你可以输入后 再删掉。
mv ssl.key xxx.key
openssl rsa -in xxx.key -out ssl.key
rm xxx.key

然后根据这个key文件生成证书请求文件
openssl req -new -key ssl.key -out ssl.csr

openssl x509 -req -days 36500 -in ssl.csr -signkey ssl.key -out ssl.crt
这里365是证书有效期