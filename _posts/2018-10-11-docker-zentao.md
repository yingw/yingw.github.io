---
layout: article
title: 用 Docker 构建禅道环境
tags: Docker
key: 2018-10-11-docker-zentao
---

# 用 Docker 构建禅道环境

## 禅道简介

>禅道是个不错的国产的开源项目管理软件，在国内很受欢迎。有分开源版、专业版、和企业版，这里介绍的是开源版

**一些地址**

- [禅道官网]( https://www.zentao.net/)
- 本文所构建的[禅道 docker 镜像](https://hub.docker.com/r/yinguowei/zentao/ )，[源码](https://github.com/yingw/docker-zentao)
- 禅道开源版 [LDAP插件](https://github.com/iboxpay/ldap) 

> 之前也有些可用的禅道开源版 docker 镜像，但是都是基于一键安装版本的，如 [lijiapengsa/zentao](https://github.com/lijiapengsa/zentao) ，本文介绍的是用源码构建方式。

**当前版本**

目前还在维护的版本有 9 和 10，7、8 版本以前虽然在公司也在用，但是基本不更新了。而且 10 版本虽然刚出没多久，但是界面完全重绘（Vue），更新发布较快。

- 9.8.3（界面比较经典的版本，在10 出来后版本9基本就不更新了）
- 10.4（目前最新版，也叫 10.4.stable）
- 还有个在 Azure 上的 [PaaS 版本](https://market.azure.cn/zh-cn/marketplace/apps/NatureEasySoft.ZentaoProjectManagementSoftware824?tab=Overview) 版本是 8，不做考虑

## 镜像构建

### Dockerfile

完整的 Dockerfile，禅道版本 10.4：

```dockerfile
FROM php:7.2.8-apache-stretch

LABEL maintainer="yinguowei@gmail.com"

ENV DEBIAN_FRONTEND noninteractive

RUN set -x \
    && apt-get -y update \
    && apt-get install -y --no-install-recommends apt-utils unzip \
    && rm -rf /var/lib/apt/lists/*

# 安装禅道需要的组件
RUN docker-php-ext-install -j$(nproc) pdo_mysql \
    && mkdir /php_session_path \
    && chmod o=rwx -R /php_session_path \
    && echo "session.save_path = \"/php_session_path\"">>/usr/local/etc/php/php.ini

ENV ZENTAO_VERSION 10.4

# 获取源码包
ADD http://dl.cnezsoft.com/zentao/$ZENTAO_VERSION/ZenTaoPMS.$ZENTAO_VERSION.stable.zip /var/www/html/

# 解压
RUN unzip /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip && rm -f /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip

RUN echo "<html>\n<head>\n<meta http-equiv=\"refresh\" content=\"0;url=/zentaopms/www/\">\n</head>\n</html>" > /var/www/html/index.html

# 备份目录挂载卷
RUN mkdir -p /var/www/html/zentaopms/tmp/backup && chmod 777 /var/www/html/zentaopms/tmp/backup
VOLUME /var/www/html/zentaopms/tmp/backup
```

#### 源码说明

- 基于官方 php 版本构建

```dockerfile
FROM php:7.2.8-apache-stretch
```

- 安装 php 插件

> 禅道要安装多个php插件有：pdo，pdo_mysql，json，filter，不过只有 pdo_mysql 默认没有装

```dockerfile
RUN docker-php-ext-install -j$(nproc) pdo_mysql
```

> 注：docker-php-ext-install 是 php 容器提供用于安装插件的工具，在 `/usr/local/bin/` 下面

- 配置 session path

> 禅道需要一个明确的存放 session 的路径配置，镜像中没有默认的 php.ini，自己生成一个（不知道为什么，之前基于 Ubuntu 的镜像里面却有一个完整的 php.ini，在目录 /usr/src/php 下）

```dockerfile
RUN mkdir /php_session_path \
    && chmod o=rwx -R /php_session_path \
    && echo "session.save_path = \"/php_session_path\"">>/usr/local/etc/php/php.ini
```

- 创建自动跳转页

> 在 Apache 项目根目录创建个 index.html，用来在没有指定上下文的时候自动重定向到 `/zentaopms/www/` 去

```dockerfile
RUN echo "<html>\n<head>\n<meta http-equiv=\"refresh\" content=\"0;url=/zentaopms/www/\">\n</head>\n</html>" > /var/www/html/index.html
```

### 编译镜像并提交

> 构建镜像并提交到 [docker hub](https://hub.docker.com/)，注意不要把任何公司相关的配置提交到镜像里

1. 先要去 docker hub 创建这个镜像的定义
2. 然后本地编译执行：
```bash
docker login
docker build -t yinguowei/zentao:10.4.stable .
docker tag yinguowei/zentao:10.4.stable yinguowei/zentao:10
docker tag yinguowei/zentao:10.4.stable yinguowei/zentao:latest
docker push yinguowei/zentao:10.4.stable
docker push yinguowei/zentao:10
docker push yinguowei/zentao:latest
```

> 提交成功后就可以在[镜像首页](https://hub.docker.com/r/yinguowei/zentao/)  发布，增加说明等

## 使用镜像

### 简单运行

```bash
docker run -d --name zentao -p 80:80 yinguowei/zentao:10.4.stable
```

运行成功后访问 [http://localhost]( http://localhost)

### 设置数据库

> 根据禅道设置向导设置项目配置，其中数据库的配置：（这里是我的windows数据库）

```php
<?php
$config->installed       = true;
$config->debug           = false;
$config->requestType     = 'GET';
$config->db->host        = '10.0.75.1';
$config->db->port        = '3306';
$config->db->name        = 'zentao';
$config->db->user        = 'root';
$config->db->password    = 'test1234';
$config->db->prefix      = 'zt_';
$config->webRoot         = getWebRoot();
$config->default->lang   = 'zh-cn';
```

**注意：**

1. 数据库 host 设置 10.0.75.1 是 docker 的宿主机网络网关地址，也是容器从容器中访问宿主服务的方式。
2. 宿主服务器上 MySQL 要设置允许外部 ip 访问，略
3. 有个小问题，登入如果报错管理员密码不对，可以试试看无痕浏览模式，可能是由于密码客户端加密后被 chrome 拦截的关系，后面去掉密码加密就好了。

### 用 docker-compose 运行集成 mysql

设置配置文件 `docker-compose.yml`

```yaml
version: '3.1'

services:
  zentao:
    image: yinguowei/zentao:10.4.stable
    restart: always
    ports:
      - "80:80"
    volumes:
      - zentao_backup_data:/var/www/html/zentaopms/tmp/backup
  mysql:
    image: mysql:5.7.23
    ports:
      - "3306:3306"
    volumes:
      - zentao_mysql_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: zentao!23
      MYSQL_DATABASE: zentao
    restart: always
volumes:
  zentao_backup_data:
  zentao_mysql_data:
```

> 注意可能需要修改 docker-compose.yml 里设定的 端口

启动

```
docker-compose up -d
```

数据库就可以直接用 mysql hostname 连接

```php
<?php
$config->installed       = true;
$config->debug           = false;
$config->requestType     = 'GET';
$config->db->host        = 'mysql';
$config->db->port        = '3306';
$config->db->name        = 'zentao';
$config->db->user        = 'root';
$config->db->password    = 'zentao!23';
$config->db->prefix      = 'zt_';
$config->webRoot         = getWebRoot();
$config->default->lang   = 'zh-cn';
```

其他命令：停止，删除

```bash
# 停止，删除
docker-compose stop
docker-compose rm
# 或者直接删除
docker-compose rm -fs
```

### Nginx 设置

建议通过 Nginx 设置反向代理到禅道的镜像地址，另外还使用了 [docker-gen](https://github.com/jwilder/docker-gen) 配置 Nginx 和 docker 更方便。

1. 启动 Nginx
2. 启动 docker-gen
3. 启动镜像时候的设置，增加 VIRTUAL_HOST 和 VIRTUAL_PORT （默认80）设置

```bash
docker run -d --name zentao -p :80 \
  -e VIRTUAL_HOST=zentao.wilmartest.cn \
  yinguowei/zentao:10.4.stable

# Nginx 重新加载配置
nginx -s reload
```

这样就可以通过 [http://zentao.wilmartest.cn](http://zentao.wilmartest.cn ) 来访问了

## 用 ECS 运行容器

TBD

## Docker Swarm 管理集群

TBD