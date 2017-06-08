---
layout:		post
title:		"内网穿透笔记"
subtitle:	"编译ngrok以及使用frp进行穿透"
date:		2017-06-09 00:02:40 +0800
author:		"Les1ie"
tags: 
  - 学习笔记
  - 内网穿透
  - 网络安全
---

## 0x00 序言
2017年6月9日00:03:07

这个也是以前的笔记

校园网有一层waf， 从外网进来的所有请求都会被掐， 而我在学校里面有好几台服务器，回到家了就不能愉快的玩耍了，于是决定搞一个内网穿透。

（其实还有一个原因是我在二基楼搭了一台mc服务器，用宽带的同学没办法连过来一起玩， 逃）

用一台二基楼里一台低配置的电脑做跳板，穿到我的公网服务器上。

最开始是用的frp做的穿透，那时候是V0.9，不知为何一直无法连接到我外网服务器，为此还去V2EX问过别人，都没好的答案。

大概等了几天发现frp更新到了V0.10, 再次尝试便成功了。
## 0x01 使用frp


更新 2017-5-23 20:22:43

突发奇想再试了一次配置frp, 发现不能用，官网给的readme里面的ssh穿透的配置文件也变了，这才发现官方发布了新版本，就在三天前，重新试了一下，问题解决

在上一个版本中，存在一个特权模式，可以让所有的配置都在client端进行，在新发型的v0.10.0版本之后，只留下了特权模式，当前所有的配置都在client端进行了。
0.10.0之前的版本不会向后兼容了。


官方下载[链接](https://github.com/fatedier/frp/releases)

```zsh
[vim server.ini]>

[common]
bind_port = 70000

$ ./frps -c server.ini
```
以及
```zsh
[vim client.ini]>

[common]
server_addr = x.x.x.x
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000


$./frpc -c ./client.ini
```

在需要ssh登录的时候只需要
```bash
$ ssh user@x.x.x.x -p6000
```

这个也可以用来转发udp链接，

当然安全起见需要在上面加一个复杂一点的token


## 0x02 使用ngrok
2017年5月29日10:25:15

### 编译多平台ngrok 和ngrokd

### 0x01 配置环境
```bash
$ git clone https://github.com/inconshreveable/ngrok.git
$ sudo apt install golang
```

### 0x02 生成密钥
```bash
$ mdir key
$ cd key
$ openssl genrsa -out rootCA.key 2048
$ openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=xx.xx.com" -days 5000 -out rootCA.pem
$ openssl genrsa -out device.key 2048
$ openssl req -new -key device.key -subj "/CN=xx.xx.com" -out device.csr
$ openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000
```
### 0x03 使用证书
```bash
$ cp rootCA.pem assets/client/tls/ngrokroot.crt
$ cp device.crt assets/server/tls/snakeoil.crt
$ cp device.key assets/server/tls/snakeoil.key
```

### 0x03 编译
编译服务端, ngorkd
```bash
$ cd ngrok
$ proxychains make release-server
```
编译客户端
```bash
$ make release-client # linux amd64 
$ GOOS=linux GOARCH=386 make release-client # 32位的windows用的
$ GOOS=darwin GOARCH=amd64 make release-client # mac的客户端， 在bin/darwin下面
$ GOOS=windows GOARCH=amd64 make release-client # win-amd64 客户端，在bin/win64下面
$ GOOS=windows GOARCH=386 make release-client # win-386客户端， 在 bin/win32下面
```




编译完成之后, ngrokd放在server端， ngrok放在客户端

### 0x04 启动

先在server端启动
```bash
$ sudo ./ngrokd -tlsKey=./key/device.key -tlsCrt=./key/device.crt -domain=xx.xx.com -tunnelAddr=:2346
```

然后在client端进入相应版本的ngrok的文件目录
新建一个ngrok.yml文件
```bash
$ cat ngrok.yml 
server_addr: "xx.xx.com:2346"
trust_host_root_certs: false

tunnels:
  ssh:
    remote_port: 222
    proto:
      tcp: 22
    
# minecraft
  mc:
    remote_port: 25565
    proto: 
      tcp: 25565

  web:
    remote_port: 55553
    proto: 
      http: 80
      
## shadowsocks
  ss:
    remote_port: 56789
    proto:
      tcp: 56789
```

接着在本地启动服务
```bash
$ ./ngrok -config=ngrok.yml start ss mc ssh
```
接着显示如下
```bash
ngrok                                                                                                                                                  (Ctrl+C to quit)
                                                                                                                                                                       
Tunnel Status                 online                                                                                                                                   
Version                       1.7/1.7                                                                                                                                  
Forwarding                    tcp://xx.xx.com:222 -> 127.0.0.1:22                                                                                              
Forwarding                    tcp://xx.xx.com:25565 -> 127.0.0.1:25565                                                                                         
Forwarding                    tcp://xx.xx.com:5678 -> 127.0.0.1:5678                                                                                        
Web Interface                 127.0.0.1:4040                                                                                                                           
# Conn                        8                                                                                                                                        
Avg Conn Time                 35977.19ms                                                                                                                               

```

windows下的配置如下

首先在对应的系统版本的ngrok文件夹
```cmd
$ more ngrok.cfg
server_addr: "xx.xx.com:4443"
trust_host_root_certs: false
```
默认的port即为4443, 如果服务器另外指定了端口，本地也需要同步修改

接在在命令行输入
```cmd
$ ngrok.exe -subdomain=a -config="ngrok.cfg" -proto=http 8080 
$ ngrok.exe -config=ngrok.cfg -proto=tcp 23451
$ cat ngrok.cfg
server_addr: "xx.xx.com:34562"
trust_host_root_certs: false
```
也可以直接把所有的参数放到配置文件中




表示转发成功，开个tmux跑，定期检查内存泄漏

2017年5月29日13:21:33

## 0x03 后记
穿透服务器是放在一台i3的电脑上，上面跑了个32bit ubuntu的虚拟机， 发现这东西真是节省内存，如果单独启动不开服务的只有40M内存的使用

最后是用了ngrok穿了ssh，ss和mc的端口出去，frp穿了ssh端口，防止其中某一个挂了我访问不到。

当然，跑服务的电脑被别人关了的话就另当别论了
