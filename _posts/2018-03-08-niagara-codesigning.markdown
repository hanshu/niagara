---
layout: post
title:  "在Niagara中如何进行Code Signing"
date:   2018-03-08 10:32:10 +0800
categories: niagara
---

## 生成Server Certificate
可以参考之前的文章[在Niagara中如何通过User Login Over SSL连接MQTT Broker](https://hanshu.github.io/niagara/niagara/mqtt/2018/02/12/niagara-with-mqtt.html)生成Root CA和用于代码签名的服务器证书。

注意：这里由于执行jarsigner时会需要alias，所以在导出pkcs12证书时可以加上`-name "alias"`选项：
```
openssl pkcs12 -export -in server.crt -inkey server.key -out server.pkcs12 -name "codesigncert"
```

从pkcs12格式证书生成keystore：
```
keytool -importkeystore -srckeystore server.pkcs12 -destkeystore server.jks -srcstoretype pkcs12
```

## Sign the jar
```
jarsigner -keystore path\to\selfcert.jks -storepass <yourPassword> path\to\jarToSign-rt.jar codesigncert
```

## Niagara configuration
由于需要在Niagara环境下运行自定义模块，所以需要把Root CA公钥证书导入到User Trust Store里，之后就可以正常运行已代码签名过的模块。

### REFLECTION permission
目前在Niagara 4.4里只有REFLECTION这个permission group需要进行代码签名：
```
<permissions>
    <niagara-permission-groups type="all">
        <req-permission>
            <name>REFLECTION</name>
            <purposeKey>.</purposeKey>
        </req-permission>

    </niagara-permission-groups>
</permissions>
```

之后就可以使用java reflection：
```
    public static <T> T fromJson(String message, Class<T> classOfT){
        T result = AccessController.doPrivileged((PrivilegedAction<T>)()->{

            return gson.fromJson(message, classOfT);
        });

        return result;
    }

    public static String toJson(Result result){

        String ret = AccessController.doPrivileged((PrivilegedAction<String>)()->{

            return gson.toJson(result);
        });

        return ret;
    }
```
    



