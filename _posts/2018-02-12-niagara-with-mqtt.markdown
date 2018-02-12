---
layout: post
title:  "在Niagara中如何通過User Login Over SSL連接MQTT Broker"
date:   2018-02-12 16:40:51 +0800
categories: niagara mqtt
---

# 在Niagara中如何通過User Login Over SSL連接MQTT Broker

## Root CA
- 生成CA Root证书的密钥：
openssl genrsa -out ca.key 2048
- 生成Root CA根证书：
openssl req -x509 -new -key ca.key -out ca.crt



## Apache Apollo (1.7.1)
### Server Certificate
- 生成服務器端證書，並用Root CA籤發
openssl genrsa -out server.key  2048
openssl req -new -key server.key  -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -days 3650 -out server.crt
- 生成pkcs12格式的证书
openssl pkcs12 -export -in server.crt -inkey server.key -out server.pkcs12
- 生成服务器端keystore
keytool -importkeystore -srckeystore server.pkcs12 -destkeystore server.jks -srcstoretype  pkcs12
### MQTT Configuration
- 按照文檔安裝部署到Apollo後，將上面生成的server.jks配置到apollo.xml：

```
<key_storage file="${apollo.base}/etc/server.jks" password="niagara" key_password="niagara"/> 
```
需要注意的是Niagara abstractMqttDriver設定的端口範圍在1-10000之間，所以需要修改Apollo默認的端口：
```
  <connector id="tcp" bind="tcp://0.0.0.0:9913" connection_limit="2000"/>
  <connector id="tls" bind="tls://0.0.0.0:9914" connection_limit="2000"/>
```

### Mosquitto Client
- 連接TCP
mosquitto_sub -t edge -p 9913 -u admin -P password
- 連接SSL
mosquitto_sub -t edge -h 192.168.1.55 -p 9914 -u admin -P password --cafile /var/lib/mqttbroker/etc/ca.crt



## Mosquitto (1.4.14)
- 如上生成服務器端證書並通過Root CA籤發
- 通過mosquitto_passwd設置用戶和密碼並創建相應的配置文件pwfile
- 在conf.d文件夾下新建自個的配置文件：
```
allow_anonymous false
password_file /etc/mosquitto/pwfile

port 8883
cafile /etc/mosquitto/ca_certificates/ca.crt
keyfile /etc/mosquitto/certs/server.key
certfile /etc/mosquitto/certs/server.crt

#tls_version tlsv1
#require_certificate true
#use_identity_as_username true
```



## Niagara abstractMqttDriver Configuration
- 按照文檔配置好MQTT Driver
- 將獲取到的Root CA公鑰證書import到User Trust Store
- 配置abstractMqttDriverDevice -> User Login Over SSL，以及相應的Username and Password



## mqtt-spy
也可以使用來進行MQTT測試，驗證是否部署配置正確。
