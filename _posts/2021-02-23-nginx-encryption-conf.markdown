---
layout: post
title:  "nginx证书加密方式调整"
date:   2021-02-23
categories: linux
---

# 背景

> 某项目上线需要进行安全检测，nginx使用之前模板配置，安全检测认为`TLS_RSA_WITH_AES_128_GCM_SHA256`等密码套件不够安全，虽然修改了nginx可以通过，但是学习了几种查看ssl支持套件的测试方法。


# 使用 openssl 检测

使用 openssl 测试是否支持ssl2加密

openssl s_client -connect example.com:443 -tls1

如果不支持 tls1 就会出现下面提示：

```
CONNECTED(00000003)
140287261328768:error:1409442E:SSL routines:ssl3_read_bytes:tlsv1 alert protocol version:ssl/record/rec_layer_s3.c:1543:SSL alert number 70
[...]
```

# 使用sslscan 检测

```
$ sslscan example.com
                   _
           ___ ___| |___  ___ __ _ _ __
          / __/ __| / __|/ __/ _` | '_ \
          \__ \__ \ \__ \ (_| (_| | | | |
          |___/___/_|___/\___\__,_|_| |_|

		sslscan version 1.10.2
		OpenSSL 1.0.2u  20 Dec 2019


Testing SSL server example.com on port 443

  Supported Client Cipher(s):
    ECDHE-RSA-AES256-GCM-SHA384
    ECDHE-ECDSA-AES256-GCM-SHA384
    ECDHE-RSA-AES256-SHA384
    [...]

  Supported Server Cipher(s):
    Failed    TLSv1  256 bits  ECDHE-RSA-AES256-GCM-SHA384
    Failed    TLSv1  256 bits  ECDHE-ECDSA-AES256-GCM-SHA384
    Failed    TLSv1  256 bits  ECDHE-RSA-AES256-SHA384
    Failed    TLSv1  256 bits  ECDHE-ECDSA-AES256-SHA384
    Rejected  TLSv1  256 bits  ECDHE-RSA-AES256-SHA
    Rejected  TLSv1  256 bits  ECDHE-ECDSA-AES256-SHA
    Failed    TLSv1  256 bits  SRP-DSS-AES-256-CBC-SHA
    [...]
```

# 使用nmap 检测
> 检测软件AppScan的检测结果和这个最相近

```
$ nmap --script ssl-enum-ciphers -p 443 example.com
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-23 21:51 CST
Nmap scan report for example.com (*.*.*.*)
Host is up (0.18s latency).
Other addresses for example.com (not scanned): *

PORT    STATE SERVICE
443/tcp open  https
| ssl-enum-ciphers:
|   TLSv1.0:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (rsa 2048) - A
|       TLS_DHE_RSA_WITH_SEED_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_SEED_CBC_SHA (rsa 2048) - A
|     compressors:
|
|     cipher preference: server
|   TLSv1.1:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA (secp256r1) - A
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA - unknown
|       TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (rsa 2048) - A
|       TLS_DHE_RSA_WITH_SEED_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_SEED_CBC_SHA (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: server
|   TLSv1.2:
|     ciphers:
|       TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (secp256r1) - A
|       TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (secp256r1) - A
|       TLS_DHE_RSA_WITH_AES_128_GCM_SHA256 (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_256_GCM_SHA384 - unknown
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA256 (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_128_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA256 (dh 2048) - A
|       TLS_DHE_RSA_WITH_AES_256_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_AES_128_GCM_SHA256 (rsa 2048) - A
|       TLS_DHE_RSA_WITH_CAMELLIA_256_CBC_SHA (dh 2048) - A
|       TLS_DHE_RSA_WITH_CAMELLIA_128_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_AES_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_256_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_AES_128_CBC_SHA (rsa 2048) - A
|       TLS_RSA_WITH_CAMELLIA_128_CBC_SHA (rsa 2048) - A
|       TLS_DHE_RSA_WITH_SEED_CBC_SHA (dh 2048) - A
|       TLS_RSA_WITH_SEED_CBC_SHA (rsa 2048) - A
|     compressors:
|       NULL
|     cipher preference: server
|_  least strength: unknown

Nmap done: 1 IP address (1 host up) scanned in 55.64 seconds
```


# nginx 部分配置

```
ssl_protocols TLSv1.2;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
```

[nginx其他部分配置参考](https://gist.github.com/palytoxin/f96d0164fec9548fa5712a7bc8ba4361)