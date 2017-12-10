## 序
因为某些原因，写了个新闻聚合的应用，取名**SCURSS** ，这里想分享的是用容器进行虚拟化的思路

## 容器
具体内容就是一个爬虫定期爬取教务处，计算机学院官网，学工部，然后检查新闻内容是否发生了变化，如果发生了变化，那么更新数据库，然后邮件推送给订阅的用户发生更新了的消息

这里想说的是用的`docker`来部署的服务， 虽然这不是什么新鲜的东西，但是我还没处理过这么多的容器之间的依赖关系，所以简单记录一下。因为这里需要的容器比较多，为了方便管理，使用了`docker-compose`来进行编排

项目的文件目录如下


```
$ tree
.
├── database
│   └── Dockerfile
├── docker-compose.yml
├── Dockerfile
├── feed_gen
├── init_send
│   └── Dockerfile
├── send_mail
├── spider
├── wait-for-it.sh
└── web
    └─ Dockerfile
```


## 容器编排
首先是写一个`docker-compose.yml`
内容如下

```
version: "3"
services:
  pgsqldb:
    image: postgres:10.1
    container_name: pgsqldb
    restart: always
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: ms17010

  scurss_web:
    build:
      context: .
      dockerfile: ./web/Dockerfile
    container_name: scurss_web
    restart: always
    ports:
      - "8000:80"
    volumes:
      - ./web/:/var/www/html/
    depends_on:
      - pgsqldb
      - init_mail_server

  mail_redis:
    image: redis
    container_name: mail_redis
    restart: always

  init_mail_server:
    build:
      context: .
      dockerfile: ./init_send/Dockerfile
    container_name: init_mail_server
    volumes:
      - ./init_send:/app
    depends_on:
      - mail_redis
      - pgsqldb
    restart: always

  manage_python:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: manage_python
    volumes:
      - .:/scurss
    working_dir: /scurss
    restart: always
    depends_on:
      - mail_redis
      - pgsqldb
    command: ["./wait-for-it.sh", "pgsqldb:5432", "--", "python", "manage.py"]

```

- `pgsqldb`，这是一个`postgres`的数据库，用来存储爬虫获取到的内容，端口映射出来方便调试，上线了之后可以去掉端口映射，直接使用容器互联，同时在环境变量中把容器的密码传了进去
- `scurss_web` 这个是web容器，最开始是用的纯php容器，后来觉得懒得麻烦，直接搞了个带apache的，把目录挂载到`/var/www/html`就能用了，当然这个潜在的问题就是整个web目录都暴露在里面了，比如`Dockerfile`就可以直接访问到，当然我可以选择不把`Dockerfile`放到这个目录, `./web/Dockerfile`内容如下

```
FROM php:5.6-apache

MAINTAINER SCURSS

RUN apt update && apt install php5-pgsql libpq-dev -y --no-install-recommends \
    && docker-php-ext-install pdo pdo_pgsql pgsql

EXPOSE 80
```

因为web需要用到`postgresql`，所以需要`php`的`psql`的支持，网上搜寻一圈找到了需要安装的包，这里只有自己写`Dockerfile`来加入这些文件了

- `init_mail_server` 这个容器是一个`flask`程序的容器，用于监听是否有用户提交订阅信息，`php`负责处理用户提交的`POST`请求然后把订阅的内容插入到数据库中，然后通过一个`get`请求通知`flask`写的一个`api`,然后`flask`程序会去数据库取出来当前的所有的用户的信息，然后去`redis`数据库中取出用户注册之前的用户列表，通过两个列表的存在差异的用户来确定哪一些是新注册的用户，然后去`postgres`中取出每一个站点的最新的十条新闻，推送到用户的邮箱，这里因为需要用到自定义的`python`包，所以自己写了`Dockerfile`, 路径是`./init_send/Dockerfile`，内容如下


```
ROM tiangolo/uwsgi-nginx-flask:python3.6

MAINTAINER scurss scurss@gmail.com

RUN pip install redis==2.10.6 arrow==0.10.0 psycopg2==2.7.3.2

```

- `mail_redis` 这个是一个`redis`的服务器，不对外开放端口，这里开放是方便我调试，上线后会关闭，不然又是一个妥妥的`redis`未授权
- `manage_python` 是项目主程序的容器，执行的入口是`python manage.py` 然后会调用一系列的模块完成新闻爬取，内容更新比较，邮件推送，生成`feed.xml`文件到`web`目录等功能，由于这个程序一旦启动就需要连接到`postgres`数据库，所以第一次我是启动失败了的，然后网上搜了下，看到了`wait-for-it.sh`这个东西，以前其实也见到过，但是不是很了解他的功能，刚好今天有这个需求，所以大概了解了一下，他是通过监听互联的容器的端口来检测服务是否启动的，如果对应的端口起来了，那么就可以判断容器是已经完成启动了，然后就会执行后面的入口程序。
  主程序的`Dockerfile`如下

```
FROM python:3.6.3-jessie

MAINTAINER SCURSS


RUN pip install psycopg2==2.7.3.2 arrow==0.12.0 redis==2.10.6 pyrss2gen==1.1 \
    lxml==4.1.1 requests==2.11.1 ipython==6.2.1 -i https://pypi.douban.com/simple

```



需求很简单，今晚就上线，怎么实现我不管


那么就用开发机吧

只要在任意的配置好了docker和docker-compose的地方，都可以直接运行整个项目
```
$ docker-compose up -d
```
所有的搭建环境的事情，我们都不需要关心，不用关心这是CentOS还是Ubuntu，或者是Debian

因为80端口不能只挂这一个服务，所以自然用了nginx来反代web,

在`/etc/nginx/conf.d/scurss.conf`

```
server {
        listen 80; server_name scurss.les1ie.com;
        access_log /var/log/nginx/scurss_access.log;
        error_log /var/log/nginx/scurss_error.log;

        location / {
                proxy_pass http://localhost:8000;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
        }
}

```

然后去`dns`里面加入一条`scurss.les1ie.com`解析到开发机

## 跋
最近事情太多了忙不过来，几个项目同时在赶工期，一天到晚敲敲敲，头晕

以后还是不要去当开发了，开发这个岗位不好呆，还是产品经理好玩

另外：

发现我真好满足，能有两个高分屏，椅子舒服，当然机械键盘也是要的，我的laji笔记本是1366*768的，看着太难受了（我真好收买，给个高分屏就行）


睡觉睡觉

2017年12月10日01:13:52




