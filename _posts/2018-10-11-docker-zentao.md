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
# 如果网络有问题就改成本地拷贝：ADD ZenTaoPMS.$ZENTAO_VERSION.stable.zip /var/www/html/
ADD http://dl.cnezsoft.com/zentao/$ZENTAO_VERSION/ZenTaoPMS.$ZENTAO_VERSION.stable.zip /var/www/html/
RUN unzip /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip && rm -f /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip

# 加上自动跳转页面  
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

> 根据需要修改 docker-compose.yml 里设定的 端口

启动

```
docker-compose up -d
```

数据库就可以直接用 `mysql` 作为 hostname 连接

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

1. 启动 Nginx 略
2. 启动 docker-gen 略
3. 启动镜像时候的设置，增加 VIRTUAL_HOST 和 VIRTUAL_PORT （默认80）设置

```bash
docker run -d --name zentao -p :80 \
  -e VIRTUAL_HOST=zentao.wilmartest.cn \
  yinguowei/zentao:10.4.stable

# Nginx 重新加载配置
nginx -s reload
```

这样就可以通过 [http://zentao.wilmartest.cn](http://zentao.wilmartest.cn ) 来访问了

## 集成 LDAP

禅道开源版默认没有集成 LDAP （专业版有），需要用插件来实现，这里使用了 插件 [iboxpay/ldap](iboxpay/ldap)

### 修改 Dockerfile

新的 Dockerfile

```dockerfile
FROM php:7.2.8-apache-stretch

LABEL maintainer="yinguowei@gmail.com"

ENV DEBIAN_FRONTEND noninteractive

RUN set -x \
    && apt-get -y update \
    && apt-get install -y --no-install-recommends apt-utils unzip libldap2-dev \
    && rm -rf /var/lib/apt/lists/*

# 安装禅道需要的组件
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
  && docker-php-ext-install -j$(nproc) pdo_mysql \
  && docker-php-ext-install ldap \
  && mkdir /php_session_path \
  && chmod o=rwx -R /php_session_path \
  && echo "session.save_path = \"/php_session_path\"">>/usr/local/etc/php/php.ini

ENV ZENTAO_VERSION 10.4

# 获取源码包
#ADD http://dl.cnezsoft.com/zentao/$ZENTAO_VERSION/ZenTaoPMS.$ZENTAO_VERSION.stable.zip /var/www/html/
ADD ZenTaoPMS.$ZENTAO_VERSION.stable.zip /var/www/html/
RUN unzip /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip && rm -f /var/www/html/ZenTaoPMS.$ZENTAO_VERSION.stable.zip

WORKDIR /var/www/html/zentaopms

# 准备工作，目录，权限
RUN touch www/ok.txt \
  && mkdir -p lib/ldap \
  && chmod 777 . \
  && chmod -R 777 lib/ldap \
  && chmod -R 777 module/user/ext \
  && mkdir -p /var/www/html/zentaopms/tmp/backup \
  && chmod 777 /var/www/html/zentaopms/tmp/backup

# LDAP 插件
#RUN curl -o module/extension/ext/ldap-master.zip https://codeload.github.com/iboxpay/ldap/zip/master \
ADD /ldap-master.zip module/extension/ext/
RUN unzip module/extension/ext/ldap-master.zip -d module/extension/ext/ \
  && mv module/extension/ext/ldap-master module/extension/ext/ldap \
  && cp -r module/extension/ext/ldap/lib/* lib/ \
  && cp -r module/extension/ext/ldap/module/* module/ \
  && mkdir -p tmp/extension/ \
  && mv module/extension/ext/ldap-master.zip tmp/extension/ldap.zip

# 加上自动跳转页面  
RUN echo "<html>\n<head>\n<meta http-equiv=\"refresh\" content=\"0;url=/zentaopms/www/\">\n</head>\n</html>" > /var/www/html/index.html

# 备份目录挂载卷
VOLUME /var/www/html/zentaopms/tmp/backup
```

**代码说明**

相比较原版，主要是安装的插件和进行一些初始设置

```bash
apt-get install -y --no-install-recommends libldap2-dev
docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/
docker-php-ext-install ldap
```

这些是运行 ldap 需要安装的环境组件和设置

```bash
touch ./www/ok.txt \
  && mkdir -p ./lib/ldap \
  && chmod 777 . \
  && chmod -R 777 ./lib/ldap
```

一些设置，ok.text 是禅道检测运行安装插件用的

下载并解压插件

```dockerfile
# LDAP 插件
RUN curl -o ./module/extension/ext/ldap-master.zip https://codeload.github.com/iboxpay/ldap/zip/master \
  && unzip ./module/extension/ext/ldap-master.zip -d ./module/extension/ext/ \
  && mv ./module/extension/ext/ldap-master ./module/extension/ext/ldap \
  && cp -r ./module/extension/ext/ldap/lib/* ./lib/ \
  && cp -r ./module/extension/ext/ldap/module/* ./module/ \
  && mkdir -p ./tmp/extension/ \
  && mv ./module/extension/ext/ldap-master.zip ./tmp/extension/ldap.zip
```

这个和官方安装插件的方式不太一样，官方推荐的插件安装方式是在启动后到后台选择安装；这里只是把插件下载后直接解压到插件对应的目录生效，效果一样，但是相对来说设置简单些。禅道用控制台安装插件大致做了这些操作：

1. 插件不解压存放在：tmp/extension 目录
2. 完整解压放在：/module/extension/ext 目录下子目录 ldap
3. 插件中 lib, module 目录复制到项目根目录 lib, module 目录
4. 往数据库表 `zt_extension` 中

上面的实现了 1~3，没有往数据库中插入，所以是不会在插件列表中看到插件，没法进行设置的

### 运行容器

```bash
docker run -d --name zentao -p :80 \
  -e VIRTUAL_HOST=zentao.wilmartest.cn \
  yinguowei/zentao:ldap
```

### 启动后设置 ldap

启动后还需要设置三个文件，参考 [禅道开源版ldap配置](https://blog.csdn.net/BigBoySunshine/article/details/80502068)

1. 修改：module/user/ext/config/ldap.php  里关于 ldap 服务器地址和认证的配置，如：

    ```php
    $config->ldap->ldap_server = 'ldap://10.229.253.35:3268';
    $config->ldap->ldap_protocol_version            = 3;
    $config->ldap->ldap_follow_referrals            = 0;  
    $config->ldap->ldap_root_dn                     = 'dc=wilmar,dc=cn';
    $config->ldap->ldap_uid_field                   = 'sAMAccountName';
    $config->ldap->ldap_bind_dn                     = 'yourdn@wilmar.cn';
    $config->ldap->ldap_bind_passwd                 = 'yourpassword';
    $config->ldap->ldap_organization               = '(objectCategory=person)(!(userAccountControl:1.2.840.113556.1.4.803:=2))';
    ```

2. 禅道登录时输入的密码会在js里加密（*md5(md5(密码+随机数))*），这样在ldap_bind()中是不知道随机数是多少的，所以会认证失败。所以要跳过加密：/module/user/js/login.js 

    ```php
    if(password.length != 32 && typeof(md5) == 'function') $('input:password').val(password);
    ```

3. 修改ldap_bind()参数

    和*ldap参数配置*里面的一样，bind_dn的格式为*user@domain,*而ldap拿到的不是这种格式，所以需要修改一下文件：  lib/ldap/ldap.class.php

    ```php
    # Attempt to bind with the DN and password
    $name = $t_info[$i]['samaccountname'][0];
    // if ( @ldap_bind( $t_ds, $t_dn, $p_password ) ) {
    if ( @ldap_bind( $t_ds, "{$name}@wilmar.cn", $p_password ) ) {
      $t_authenticated = true;
      break;
    }
    ```

容器内不可以编辑文件的话，把文件在外面编辑好拷贝进去
```bash
docker cp ldap.class.php zentao:/var/www/html/zentaopms/lib/ldap/ldap.class.php
docker cp login.js zentao:/var/www/html/zentaopms/module/user/js/login.js
docker cp ldap.php zentao:/var/www/html/zentaopms/module/user/ext/config/ldap.php
```

注意这些文件修改的时候都不要直接在进行内修改，尽量将修改过程脚本化下来，后面章节还会介绍用 sed 命令，且不要把自己公司的账号地址发布到镜像中去

重启下 apache ，在容器内 `service apache2 restart` ，会自动把容器停掉，然后重启容器就行了 `docker start zentao`（也有可能不需要这步重启）

接下来就可以用 ldap 后 AD 用户登入，以及 admin 用户本地登入了。

### 可配置化

上面的做法是把配置文件在外面设置好拷贝覆盖，但是按照十二范式的推荐，首先过程没有脚本，其次代码不可幂等，还是建议将配置参数化并自动写入配置文件

一共新增以下环境变量（等号右边的既是默认值）

```bash
export LDAP_HOST=10.229.253.36
export LDAP_PORT=389
export LDAP_ROOT_DN=dc=wilmar,dc=cn
export LDAP_UID_FIELD=sAMAccountName
export LDAP_BIND_DN=yourdn@wilmar.cn
export LDAP_BIND_PASSWORD=yourpassword
export LDAP_DOAMIN=wilmar.cn
```

Dockerfile 里设置

```dockerfile
ENV LDAP_HOST=10.229.253.36
ENV LDAP_PORT=389
ENV LDAP_ROOT_DN=dc=wilmar,dc=cn
ENV LDAP_UID_FIELD=sAMAccountName
ENV LDAP_BIND_DN=yourdn@wilmar.cn
ENV LDAP_BIND_PASSWORD=yourpassword
ENV LDAP_DOMAIN=wilmar.cn
```

一开始尝试将环境变量值写到 PHP，后来发现 RUN 命令是在镜像编译时写进去的环境变量默认值，所以要改成在 PHP 内获取环境变量

（以下是环境变量值写入，作废）
```bash
sed -i 's/^\$config->ldap->ldap_server.*$/\$config->ldap->ldap_server = '\''ldap:\/\/'$LDAP_HOST':'$LDAP_PORT\'';/' ldap.php
sed -i 's/^\$config->ldap->ldap_root_dn.*$/\$config->ldap->ldap_root_dn = '\'$LDAP_ROOT_DN\'';/' ldap.php
sed -i 's/^\$config->ldap->ldap_uid_field.*$/\$config->ldap->ldap_uid_field = '\'$LDAP_UID_FIELD\'';/' ldap.php
sed -i 's/^\$config->ldap->ldap_bind_dn.*$/\$config->ldap->ldap_bind_dn = '\'$LDAP_BIND_DN\'';/' ldap.php
sed -i 's/^\$config->ldap->ldap_bind_passwd.*$/\$config->ldap->ldap_bind_passwd = '\'$LDAP_BIND_PASSWORD\'';/' ldap.php
sed -i 's/#\$config->ldap->ldap_organization/\$config->ldap->ldap_organization/' ldap.php
```

>注：最后一行是默认启用只检测激活用户，不需要可以手动注释掉

Dockerfile 里三个文件的更新写法：

```dockerfile
# ldap.php
RUN sed -i 's/^\$config->ldap->ldap_server.*$/\$config->ldap->ldap_server = '\''ldap:\/\/'\'' \. getenv('\''LDAP_HOST'\'') \. '\'':'\'' \. getenv('\''LDAP_PORT'\'');/' module/user/ext/config/ldap.php \
  && sed -i 's/^\$config->ldap->ldap_root_dn.*$/\$config->ldap->ldap_root_dn = getenv('\''LDAP_ROOT_DN'\'');/' module/user/ext/config/ldap.php \
  && sed -i 's/^\$config->ldap->ldap_uid_field.*$/\$config->ldap->ldap_uid_field = getenv('\''LDAP_UID_FIELD'\'');/' module/user/ext/config/ldap.php \
  && sed -i 's/^\$config->ldap->ldap_bind_dn.*$/\$config->ldap->ldap_bind_dn = getenv('\''LDAP_BIND_DN'\'');/' module/user/ext/config/ldap.php \
  && sed -i 's/^\$config->ldap->ldap_bind_passwd.*$/\$config->ldap->ldap_bind_passwd = getenv('\''LDAP_BIND_PASSWORD'\'');/' module/user/ext/config/ldap.php \
  && sed -i 's/#\$config->ldap->ldap_organization/\$config->ldap->ldap_organization/' module/user/ext/config/ldap.php

# login.js
RUN sed -i 's/md5(md5(password) + rand)/password/' module/user/js/login.js

# ldap.class.php 
# $t_dn = $t_info[$i]['dn']; 改为：  $t_dn = "{$t_info[$i]['samaccountname'][0]}@{$_ENV['LDAP_DOMAIN']}";
RUN sed -i 's/$t_dn =.*$/$t_dn = $t_info[$i]['\''samaccountname'\''][0] . "@" . $_ENV['\''LDAP_DOMAIN'\''];/' lib/ldap/ldap.class.php
```

注意几点
1. sed 里表达式中特殊字符需要转义，如 \/ \. \$
2. 有个特殊的单引号要放到字符串外面再用 \' 转义，所以看上去是 '\''
3. 变量可以在外面用 $ 引用
4. 字符串拼接用 . 或者 `"{$xxx}"` 方式获取变量
5. 环境变量用 `getenv('xxx')` 或者 `_EVN['xxx']` 获取，不能用 `$_SERVER['xxx']`

用参数运行例子：

```bash
docker run -d -p 8080:80 --name=zentao -e LDAP_BIND_DN=yourdn@wilmar.cn -e LDAP_BIND_PASSWORD=yourpassword yinguowei/zentao:ldap
```

完整的例子

```bash
docker run -d -p 8080:80 --name=zentao \
  -e LDAP_HOST=10.229.253.36 \
  -e LDAP_PORT=389 \
  -e LDAP_ROOT_DN=dc=wilmar,dc=cn \
  -e LDAP_UID_FIELD=sAMAccountName \
  -e LDAP_BIND_DN=yourdn@wilmar.cn \
  -e LDAP_BIND_PASSWORD=yourpassword \
  yinguowei/zentao:ldap
```

### 插件缺陷

最后，这个 ldap 并没有做到完善，还存在以下几个问题

1. 只做了登入验证，在系统内其他管理功能当需要输入管理密码的时候，如果管理员是 AD 用户，这时候是不会用设置的 AD 认证的，还是初始化用户的那个管理员密码，可以勉强适用，比如管理员建议保留原来的 admin（用户名不要改），或者个别 AD 账号设置管理员，同时用管理设置其在禅道内账号的密码和 AD 一致
2. 没有界面配置，毕竟免费的
3. 不支持多域
4. 首先需要把客户端加密的代码去掉，对 AD 用户影响不大，数据库用户不安全，建议不要用
5. domain 的地址除了配置还需要写到 ldap_bind() 函数里，代码和配置渗透


## 用 ECS 运行容器

很简单，基本就是把 Dockerfile 往 ECS 推

## Docker Swarm 管理集群

TBD
