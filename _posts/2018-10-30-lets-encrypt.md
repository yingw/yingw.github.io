---
layout: article
title: 用 Let's Encrypt 配置网站 HTTPS
tags: Nginx
key: 2018-10-30-lets-encrypt
---

# 用 Let's Encrypt 配置网站 HTTPS

## Let's Encrypt 介绍

[Let's Encrypt](https://letsencrypt.org/) 是一家免费提供 SSL 证书签发的 CA 机构，旨在推进网站从 HTTP 向 HTTPS 过度的进程。

主要优缺点有：

- 永久免费
- 申请简单：设计了 ACME 协议，可以用客户端自动即可申请，没有人工过程，目前官方推荐的客户端是 [certbot](https://certbot.eff.org/)
- 有许多大厂支持，Mozilla、思科、Akamai、IdenTrust、Facebook 和 EFF 等，稳定有效
- 2018 年 3 月推出了 ACMEv2 协议，开始支持通配符证书（泛域名）申请方式
- 获得 IdenTrust 交叉签名，可以被 Mozilla、Google、Microsoft 和 Apple 等主流的浏览器所信任
- 缺点：90 天有效期，需要定期续约，不过也很简单

## 准备工作

题外话，一些准备工作

1. 买域名：很多途径 [万网](https://wanwang.aliyun.com/)、[阿里云域名交易](https://mi.aliyun.com/)、[GoDaddy](https://sg.godaddy.com/zh/) 都可以，这里用一个 [http://www.oou.fun]() 做例子
1. 备案：懂的，在国内需要备案才能正常访问。 [阿里云备案服务](https://beian.aliyun.com/)，整个过程三到五天，需要拍照、打印、上传、等待

## 安装 certbot

环境：
- CentOS 7
- Nginx

根据 certbot 官网的[安装说明](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx) 进行安装，会有一堆依赖版本问题。安装步骤

1. 需要安装 EPEL
    ```
    sudo yum install epel-release
    ```
2. 安装 Certbot
    ```
    sudo yum install python2-certbot-nginx
    ```

这样应该就装好了，但是实际运行起来会有多个问题：

1. 报错 `urllib3` 版本太低。可以卸载后重新安装
    ```
    sudo pip uninstall urllib3
    sudo yum install python-urllib3
    ```
2. 报错 `phOpenSSL` 版本不匹配。可以手动下载新版本安装，[参考](https://github.com/certbot/certbot/issues/4514#issuecomment-296886145)
    ```
    wget http://mirrors.163.com/centos/7.5.1804/cloud/x86_64/openstack-ocata/common/pyOpenSSL-0.15.1-1.el7.noarch.rpm
    sudo rpm -Uvh pyOpenSSL-0.15.1-1.el7.noarch.rpm
    sudo yum install certbot

    sudo yum -y install yum-utils
    sudo yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional
    ```

所以后来改成从 GitHub 获取，就一次搞定了，第一次运行会安装所有依赖：

```
git clone https://github.com/certbot/certbot.git --depth=1
 ./certbot-auto ...
```

## 生成证书

一些参数说明
- `certonly`：只创建证书
- `-d 域名`：要注册哪个域名，可以多个 `-d "*.oou.fun" -d oou.fun`
- `--email` 注册的 email 地址：需要有个 email 来注册，可以在命令中直接指定了 `--email yinguwei@gmail.com`
- `--agree-tos` 同意协议：可以在过程中 Y，也可以命令中直接同意
- `--server` 注册地址，默认是 V1，V2 好像也不需要指定 `--server https://acme-v02.api.letsencrypt.org/directory`
- `--webroot -w /usr/local/nginx/cert` 证书直接过去

详细的[参数说明](https://certbot.eff.org/docs/using.html)

一些例子：

```bash
./certbot-auto certonly --standalone --email yinguowei@gmail.com -d oou.fun -d www.oou.fun --agree-tos
```

泛域名方式：

```bash
./certbot-auto --server https://acme-v02.api.letsencrypt.org/directory -d "*.xxx.com" --manual --preferred-challenges dns-01 certonly
```

指定server（非必要）
```bash
certbot certonly --preferred-challenges dns --manual -d *.yourdomain.com --server https://acme-v02.api.letsencrypt.org/directory
```

```bash
certbot-auto certonly  -d *.newyingyong.cn --manual --preferred-challenges dns
```

另外有个高级的 acme.sh 客户端可以自动：
- https://blog.csdn.net/kikajack/article/details/80408145
- https://github.com/Neilpang/acme.sh

### 我的最终版
```bash
./certbot-auto certonly -d "*.oou.fun" -d oou.fun --manual --preferred-challenges dns --email yinguwei@gmail.com --agree-tos
```

会提示到域名里面设置个 txt

```bash
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.oou.fun with the following value:

RPE9M1AX0O746u30nxtcoZUDW95R88SzaspjQZmjWBg

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name
_acme-challenge.oou.fun with the following value:

Oj_WlJ_DQ5oGAvt3rVznRQN_fEcyXkmAf9vGmmHnPJ4

Before continuing, verify the record is deployed.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

```

要到 阿里云后台云解析DNS 里面设置新解析规则，而且中间还要再变一次

提示修改内容时不要急，先测试域名解析是否生效可以用 Linux 工具 dig，成功解析到了再让 certbot 继续验证：

```bash
yum install bind-utils
dig _acme-challenge.oou.fun txt
dig -t txt _acme-challenge.newyingyong.cn @8.8.8.8 
```

然后就生成成功了

```bash
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/oou.fun/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/oou.fun/privkey.pem
   Your cert will expire on 2019-02-05. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot-auto
   again. To non-interactively renew *all* of your certificates, run
   "certbot-auto renew"
```

### 其他命令

查看证书

```bash
./certbot-auto certificates
```

校验证书
```bash
openssl x509 -in /etc/letsencrypt/archive/oou.fun/cert1.pem -noout -text
```

三个月内更新证书

```bash
certbot-auto renew
```

但是如果是使用了泛域名方式申请的证书，还需要指定手动方式并提供身份认证，如果是阿里云的服务器可以用一个 python 的[脚本](https://github.com/ywdblog/certbot-letencrypt-wildcardcertificates-alydns-au) 结合在阿里云申请的 accesskey 来完整更新，修改里面的 Key Id 和 Secret：

在阿里云的域名配置里面把 `_acme-challenge` 这个设置删掉或改名空出来，然后执行：

```bash
./certbot-auto renew --cert-name oou.fun --manual-auth-hook /root/certbot-letencrypt-wildcardcertificates-alydns-au/python-version/au.sh
```

就会发现类似在申请时一样的在阿里云上生成这个配置并通过认证。

最好设置 Nginx 重新加载，或者设置自动定时更新

```bash
30 1 10 * * /usr/bin/certbot renew && /usr/sbin/nginx -s reload # 每月10日1点30分执行一次
```

## Nginx 配置

接下来要把证书配置到 Nginx，原来的配置文件：`/etc/nginx/confg.d/default.conf`:

```conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

在 /etc/nginx/confg.d/ 下面新建一个 `ssl.conf`，指定证书地址，开启 ssl 和 443 端口

```conf
server {
    server_name oou.fun;
    listen 443 http2 ssl;
   # ssl on;
    ssl_certificate /etc/letsencrypt/archive/oou.fun/fullchain1.pem;
    ssl_certificate_key /etc/letsencrypt/archive/oou.fun/privkey1.pem;
    ssl_trusted_certificate  /etc/letsencrypt/archive/oou.fun/chain1.pem;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```

重新加载使配置生效

```
nginx -s reload
```

就好了。证书信息
![image](https://note.youdao.com/yws/public/resource/2579c2414545ba518d1194a3c39cf415/xmlnote/6508DE5CB1B14B8CAF25F32DD3B7AA39/44313)

### 限制只用 HTTPS 访问

如果还想要仅限 https ，可以用 rewrite

default.conf
```conf
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name oou.fun;
    rewrite ^(.*) https://$server_name$1 permanent;
}
```

## 附：DNS 解析相关设置

![image](https://note.youdao.com/yws/public/resource/2579c2414545ba518d1194a3c39cf415/xmlnote/C9E2303ED84D47339555996D30486F39/44468)