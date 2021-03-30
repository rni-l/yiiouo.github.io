---
title: ngrok配置
date: 2018-10-08 09:00:00
tags: ["linux", "ngrok"]
categories: ["记录"]
draft: true
---
### 服务端配置

生成证书

```
openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=ngrok.rni-l.com" -days 5000 -out rootCA.pem
openssl genrsa -out device.key 2048
openssl req -new -key device.key -subj "/CN=ngrok.rni-l.com" -out device.csr
openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

```

替换证书

```
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp device.crt assets/server/tls/snakeoil.crt
cp device.key assets/server/tls/snakeoil.key
```

编译服务端： `make release-server`

编译客户端（我的是 mac，执行的命令不一样）：`GOOS=darwin GOARCH=amd64 make release-client`

开启服务

`./bin/ngrokd -domain="ngrok.rni-l.com" -httpAddr=":8088" -httpsAddr=":8089"`



通过 scp 下载

`scp root@112.74.168.130:/root/go/src/github.com/inconshreveable/ngrok.zip /Users/apple/D/document `



### 客户端

建立配置文件 `ngrok.cfg`

```
server_addr: “ngrok.morongs.com:4443"
trust_host_root_certs: false

```

开启服务：

`./bin/darwin_amd64/ngrok -subdomain demo -config=./ngrok.cfg 1200 `



