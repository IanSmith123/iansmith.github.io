---
layout:		post
title:		"cpu 使用情况监控"
subtitle:	"需求是第一生产力"
date:		2018-09-29 20:40:15 +0800
author:		"Les1ie"
catlog:   true
tags: 
  - 运维
  - minecraft
---
# 序
influxdb+grafana 监控CPU使用情况

# 需求

想看看玩minecraft的时候电脑CPU和服务器CPU使用率的曲线到底是什么样的, 怎么这么卡


很早以前便听说过influxdb时序数据库， grafana仪表盘，最近玩MC的时候电脑风扇似乎很累，牛羊稍微养多一点就开始卡服了，一直开着glances看cpu使用似乎不是很靠谱，并且只能看最近很短一段时间内的，而且是均值，所以决定动手做一个监控



当我怀揣着想法上网搜解决方案的时候，发现我能想到的别人早就想到了 :)   (这一条果然是永恒的真理) 很多用这两个组合显示数据的例子


不过自己动手，丰衣足食，最后我还是花了一个多小时把这个demo弄出来了


效果如图(效果图不是玩MC的时候截的)

![](http://static.scuseek.com/20180929-201339.png)


![](http://static.scuseek.com/20180929-202458.png)



绿色是笔记本，黄色是服务器 :)

![](http://static.scuseek.com/20180929-234541.png)



# 需要的环境

- influxdb
- grafana
- docker


# 使用方法

1. 存数据库的服务器上
   ```
   docker-compose up -d
   ```

2. 需要监控的服务器上 

   ```bash
   pip3 install -r requirements.txt
   python3 main.py
   ```
   为了方便部署，把封装的文件合成了一个文件，小脚本还是怎么方便怎么来 :) 下面这个脚本更方便一点
   ```
   python3 get-usage-and-upload.py
   ```
3. grafana默认账户密码admin admin, 登录进去鼠标点点点点点点点设定data source 
![](http://static.scuseek.com/20180929-202723.png)

4. 添加折线图设定一番即可
   ![](http://static.scuseek.com/20180929-202635.png)


# 缺点

本来就带不动游戏了，现在还需要带influxdb和grafana, 可行的解决方案是把这俩货放在其他服务器上，把clinet丢到mc服上面。

# 跋

开源万岁，感谢这些开源项目的作者。



需求是第一生产力？



代码见github, 仓库地址 [https://github.com/IanSmith123/cpu-monitor](https://github.com/IanSmith123/cpu-monitor)



话说以前福星老师让我做这个的时候咋个那么久都没弄出来，说到底还是太菜了



Les1ie



2018年9月29日20:16:25



# refer:

- https://github.com/influxdata/influxdb-python

- https://github.com/nicolargo/docker-influxdb-grafana/blob/master/docker-compose.yml