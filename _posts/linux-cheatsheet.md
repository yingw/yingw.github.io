# Linux Cheatsheet

- Redhat 系列：RedHat、CentOS、Fedora
- Debian 系列：Debian、Ubuntu、Deepin、Elementary OS、Mint
- Arch 系列：Arch、Manjaro

推荐配置
- 开发环境：Windows + Hyper-V (Ubuntu) or Deepin
- 生产环境：CentOS
- Docker

桌面：Cinnamon, GNOME, KDE Plasma, LXDE, LXQt, MATE, Xfce

>以下都以 CentOS 7+ 版本为基础

### 更新国内源

查看安装过软件包
```
rpm -qa|grep fastestmirror
```
测试下
```
yum check-update
```

会提示加载 fastestmirror 插件

如果都太慢，百度：CentOS 镜像
网易镜像：http://mirrors.163.com/.help/centos.html
或者阿里：https://opsx.alibaba.com/mirror

## yum

-i install
-y
-iUvh ?

## EPEL
[EPEL](http://fedoraproject.org/wiki/EPEL)是centos等衍生发行版，用来弥补centos内容更新有时比较滞后或是一些扩展的源没有。
`yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm` or `yum install -y epel-release`
还可以直接下载到 yum 目录
`wget -O /etc/yum.repos.d/epel-7.repo http://mirrors.aliyun.com/repo/epel-7.repo`


## centos 升级

```
sudo yum -y update
sudo yum -y upgrade
```

### 有冲突
yum --skip-broken upgrade
yum check （时间很长 CPU 100）

yum install yum-utils
yum clean all
yum-complete-transaction --cleanup-only


```
package-cleanup --cleandupes
```

package-cleanup --problems

rpm -q xxx
rpm -e xx
yum remove xx
yum update xx
## deepin 系统升级

普通升级软件
```
sudo apt-get -y update
sudo apt-get -y upgrade
```

有新发行版本升级时
```
sudo apt-get -y update
sudo apt-get -y dist-upgrade
```

查看版本
`lsb_release -a` 或 `screenfetch`

## cache
```
yum clean all
yum makecache fast
```

## 常用工具

```
yum install -y wget vim lsb git zsh zip unzip lrzsz screen bash-completion telnet net-tools mod_ssl openssl
```
gcc-c++ screen zsh *oh-my-zsh

安装 chrome ?

### oh-my-zsh
http://ohmyz.sh/
```
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

Theme:
```
vim ~/.zshrc
ZSH_THEME="agnoster"
```
修改默认shell

chsh -s $(which zsh)
echo $SHELL
cat /etc/shells 
zsh --version
改回bash

chsh -s $(which bash)
### chrome headless ?


### 安装 JDK、Java 环境
```
yum -y remove java-1.7.0-openjdk*
yum -y remove java-1.8.0-openjdk*
yum -y install java-1.8.0-openjdk-devel
```

删除内置 MySQL `yum erase mysql`
安装 MySQL
```
wget https://repo.mysql.com/mysql57-community-release-el7-11.noarch.rpm
rpm -Uvh mysql57-community-release-el7-11.noarch.rpm
yum install mysql-community-server
```

## 网卡

启动网卡
```
# cd /etc/sysconfig/network-scripts
# ls ifcfg-* | grep -v ifcfg-lo
ifcfg-eno1
# ifup eno1
```

ip addr
ifconfig (install net-tools)
修改/etc/sysconfig/network-scripts/ifcfg-eno1配置文件，将其中的ONBOOT=no改为ONBOOT=yes来实现网卡的自动启动。

## 修改 host
加一行
```
echo "10.229.15.96 devops.wilmar.cn yinpc" >> /etc/hosts
```

## 其他
sudo apt-get install sysv-rc-conf
sudo sysv-rc-conf
图形化表格管理初始化脚本 "/etc/rc{runlevel}.d/"

查看 CPU 信息
```
cat /proc/cpuinfo
```


物理 CPU 数
```
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
```

核数
```
cat /proc/cpuinfo | grep "core id" | sort | uniq | wc -l
```

逻辑 CPU 数
```
cat /proc/cpuinfo | grep "processor" | sort | uniq | wc -l
```
1U 双核 4线程


free -m

top

yum install nmon

查看内核 jessie

- 查看所有版本`lsb_release -a`
- 内核版本`cat /proc/version`
- 发行版本`cat /etc/issue`
- 内核版本`uname -a`
- 简短版本`uname -r`
- `lsb_release -cs` -> jessie

查看桌面版本
echo $DESKTOP_SESSION

apt-get install screenfetch
screenfetch -s 截屏

## screenFetch
```
git clone https://github.com/KittyKatt/screenFetch.git
cd screenFetch
./screenfetch-dev
```
或这样装
```
wget https://github.com/KittyKatt/screenFetch/archive/master.zip
unzip master.zip
sudo mv screenFetch-master/screenfetch-dev /usr/bin/screenfetch
```
还有个 `yum install linux_logo`


防火墙
[iptables](https://wiki.debian.org/iptables)
iptables --list
iptables -L
ufw
firewalld

端口占用

lsof -i:3306

图形 Gufw

chkconfig
/sbin/iptables -I INPUT -p tcp --dport 8080 -j ACCEPT


firewall-cmd --add-port=8080/tcp --permanent --zone=public
firewall-cmd --reload

清除某个包的缓存
apt-get purge libmysqlclient18

## oh-my-zsh
[各种快捷键](https://github.com/robbyrussell/oh-my-zsh/wiki/Cheatsheet)

`bindkey` 直接查看快捷键
```
"^A" beginning-of-line
"^E" end-of-line
```

简易查进程
pgrep mysqld

pkill


源码安装
1. ./configure
2. make
3. make install

卸载
make unistall xxxx

## alias 别名
vim ~/.bash_profile
`alias node="nodejs"`
:wq

. ~/.bash_profile


users 当前用户

## apt
apt-cache search git | grep ^git

## 解压
tar zxvf
cxvf

解压 tar.xz
xz -d file.tar.xz
Or
tar xvJf  file.tar.xz


###### 释放 Cache Buffer

```
# sync && echo 3 > /proc/sys/vm/drop_caches
```

## 用户管理
新加一个用户，分配组
```
useradd yinguowei -g wheel
passwd yinguowei
```
wheel 好像默认在 sudoer 组里

把用户加入、移出组？

加入 sudoer 组
也可以用命令 `visudo`（不过不太会用）
然后编辑加上

```
yinguowei	ALL=(ALL)	NOPASSWD: ALL
```

或者
```
echo "yinguowei ALL=(ALL) ALL" >> /etc/sudoers
```

修改密码

```
passwd
```

- 用户清单 `/etc/passwd`
- 组 `/etc/group`
- 密码保存在 `/etc/shadow`

zse4rfvgy7,./

切换到 root `sudo -i`
切换到特定用户 `su - [login]`

加 `useradd yinguowei` 会自动指定 home 路径

userdel -r yinguowei
删除用户，且自动删除用户目录

```
usermod -a -G sudo caoshengqing
```

/etc/sudoers
添加用户到sudo
yinguowei ALL=(ALL) ALL
添加组
%wheel ALL=(ALL) ALL

chmod +w /etc/sudoers

```
sudo -i
useradd yinguowei
passwd yinguowei
cp /etc/sudoers /etc/sudoers.bak
chmod +w /etc/sudoers
echo "yinguowei ALL=(ALL) ALL" >> /etc/sudoers
chmod -w /etc/sudoers
```

## 增加记住 sudo 命令密码时间
默认是 5 分钟，修改 `/etc/sudoers`
找到 `Defaults	env_reset` 后面增加 `, timestamp_timeout=600` 600 分钟。
`Defaults	env_reset, timestamp_timeout=600`


## 配置 VNC

需要桌面

## 文件权限
增加、收回权限
chmod +w file
chmod -w file

## 下载
默认 curl 是终端显示，wget 是下载
```
curl -o <url>
wget -O <url>
```

curl -x ...

## 服务

注册

启停

service docker start
开机自启
chkconfig docker on
Or
systemctl enable/start/stop xxx.servcie

常用服务：
mysqld
firewalld
ssh?

## 防火墙

### iptables VS firewalld

iptables -S|grep docker
iptables -I INPUT 4 -i docker0 -j ACCEPT

```
firewall-cmd --add-port=3306/tcp --permanent --zone=public
firewall-cmd --reload
```
iptables --list

Show IPs on the site/server at the moment:
```
# netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -n
```

## hostname

显示hostname `hostname` or `hostnamectl`
修改hostname（临时，重启失效）`hostname test.wilmar.cn`
修改hostname（永久）`echo "HOSTNAME=test.wilmar.cn" >> /etc/sysconfig/network`
还有个地方
```
echo "test.wilmar.cn" > /etc/hostname
echo "127.0.0.1 test.wilmar.cn" >> /etc/hosts
```
查看`hostnamectl --pretty set-hostname test.wilmar.cn`

## 设置时区

ls -la /etc/localtime
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
date

## Sed

## grep

从文件中找
grep 'password' /var/log/mysqld.log


## 远程桌面
yum install tigervnc-server
cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
## 压缩解压

```
$ tar -Jxvf node-v6.10.0-linux-x64.tar.xz
$ tar -zxvf eclipse-jee-neon-2-linux-gtk-x86_64.tar.gz
```
unzip file.zip
解压到指定
unzip file.zip -d dir
jar -cvfM0 Test.jar ./

## ln 软连接

```
ln -s /opt/nhs/apache-tomcat-8.0.52/bin/startup.sh ~/startup.sh
```

## alias

在 /etc/bashrc 或者 ~/.bashrc 中加上一行

```
alias logs='tail -f /opt/nhs/apache-tomcat-8.0.52/logs/catalina.out'
```

再执行
```
source ~/.bashrc
```
生效

## xrdp 远程
```
yum install tigervnc-server
```
开机自启
```
service vncserver start
chkconfig vncserver on
```

Ubuntu 可以直接
```
apt-get install xrdp
```
CentOS [参考](https://blog.csdn.net/xiaohai928ww/article/details/53081159)

## Md5

md5sum file
可以把结果写到文件
md5sum file > file.md5
再从文件读取比对
md5sum -c file.md5

还有：sha512sum
















# Word 文档 Wilmar Linux Cheatsheet 1.0




Linux Cheet Sheet for Wilmar
殷国伟、张勉、潘鑫冬、张健、王闯、胡章艮 2017-1

## 目录
Linux Cheet Sheet for Wilmar	1
1	系统工具	4
1.1	关机&重启	4
1.	shutdown	4
1.2	进入多个终端tty	4
1.3	进入桌面环境	4
2.	startx	4
2	用户管理	5
2.1	查看当前用户	5
3.	users, w, who	5
4.	id, group	5
5.	用户信息配置文件	5
2.2	用户管理	5
6.	adduser	5
7.	passwd	5
8.	su	6
9.	sudo	6
3	文件处理类	6
3.1	列出文件	6
10.	ls	6
11.	pwd	7
3.2	文件操作	7
12.	mkdir	7
13.	rmdir	7
14.	cp	7
15.	mv	7
3.3	权限设置	8
16.	chmod	8
17.	chown	8
18.	特殊目录符号	8
3.4	查找文件	8
19.	find	8
20.	locate	9
3.5	查看文件	9
21.	tail	9
22.	cat	9
23.	head	9
24.	od	9
3.6	文本编辑工具	9
25.	vi/vim	9
26.	nano	11
27.	emacs	11
28.	less	11
29.	more	11
4	系统设置	11
4.1	防火墙配置	11
30.	iptables (CentOS6)	11
31.	firewall-cmd（CentOS7）	11
32.	ln	12
4.2	挂载/卸载	12
33.	mount	12
34.	unmount	12
4.3	安装包	12
35.	yum (RHEL/CentOS)	12
36.	rpm (RHEL/CentOS)	12
37.	apt-get (Debian/Ubuntu)	13
38.	dpkg (Debian)	13
4.4	网络设置	13
39.	ifconfig	13
40.	设置hosts	13
41.	DNS设置	13
42.	host	13
43.	traceroute	14
4.5	执行周期任务	14
44.	cron	14
45.	crontab	14
46.	zeus	14
4.6	磁盘信息	14
47.	df	14
48.	du	15
4.7	内存管理	15
49.	free	15
50.	ps	15
51.	kill	15
52.	pstree	15
53.	top	15
4.8	查看端口	16
54.	netstat	16
55.	lsof	16
56.	bc	16
4.9	后台/屏幕管理	16
57.	screen	16
58.	Ctrl + z	17
59.	&	17
60.	nohup	17
61.	jobs	17
62.	fg %1	18
63.	bg %1	18
64.	kill %1	18
4.10	系统服务	18
65.	systemctl（CentOS7）	18
5	第三方工具	18
5.1	FTP	18
66.	scp	18
67.	ssh	18
68.	ftp	19
69.	make	19
5.2	压缩/解压	19
70.	tar	19
71.	zip/unzip	19
72.	gzip FILE	20
5.3	下载工具	20
73.	wget	20
74.	curl	20
75.	telnet	20
76.	lsof	20
5.4	源码编译	20
77.	gcc	20
6	其他	21
6.1	信息查询	21
78.	lscpu	21
79.	uptime	21
80.	uname -a	21
81.	date	21
82.	cal	21
83.	whoami	21
84.	which	21
85.	whereis	22
86.	man COMMAND	22
87.	info COMMAND	22
6.2	管道符	22
88.	|	22
89.	grep	22
90.	sort	22
91.	cpio	22
92.	>	23
6.3	其他的其他	23
93.	CTRL-c	23
94.	CTRL-z	23
95.	sed	23
96.	echo	23
97.	clear	23
98.	touch file	23
99.	ping IP/hostname	23
6.4	设置环境变量	23


##	系统工具
1.1	关机&重启
1.	shutdown
```
# shutdow -r/-h now
```
重启/关机
```
# shutdown –r +10
```
十分钟后重启
```
# shutdown -r 10:00
```
10点重启
```
# init 0 
```
1.2	进入多个终端tty
```
Ctrl+Alt+F1~F6
```
1.3	进入桌面环境
2.	startx
```
# startx
```
##	用户管理
2.1	查看当前用户
3.	users, w, who
```
$ users
$ w
$ who
```
4.	id, group
```
$ id
$ groups
```
查看用户id和组

5.	用户信息配置文件 

记录用户权限 `/etc/passwd`

记录用户密码 `/etc/shadow`

## 用户管理
6.	adduser
添加用户
```
$ adduser USER
```
7.	passwd
修改密码
```
$ passwd username
```
8.	su
切换到user，默认root
```
$ su - user
```
9.	sudo
Run as root group
```
$ sudo COMMAND
```
配置文件 `/etc/sudoers`

切换至root用户
```
$ sudo - i
```

TODO: echo >> suders, sh -c ...

## 文件处理类
3.1	列出文件
10.	ls
```
$ ls
$ ls /home
```
列出文件，默认当前目录
```
$ ls -a
```
包括隐藏文件（以.开头的文件）
```
$ ls -l
```
以列表格式查看，或$ ls -al
```
$ ll
```
别名 alias TODO: 同上
```
$ ls -rt
```
按时间排序查看
```
$ ls -Sh
```
按大小排序
```
$ ls -Srh
```
倒序
```
$ ls -lSr |more 
```
或者使用sort
```
$ ll -h |grep '^[^d]' |sort -n
```
11.	pwd
显示当前目录（print working directory）
```
$ pwd
```
3.2	文件操作
12.	mkdir
创建文件夹
```
$ mkdir DIRNAME
```
连续创建目录树
```
$ mkdir -p /dir1/dir2/dir3 
```
13.	rmdir
删除空目录
```
$ rmdir DIRNAME
```
删除目录（强制、循环）
```
$ rm -rf DIR
```
删除后发现大小没变，是被其他进程占用了，查看：
```
$ lsof |grep deleted
```

删库跑路（三思而后行！！！）
```
sudo rm -rf /*
```
14.	cp
复制
```
$ cp FILE1 FILE2
```
15.	mv
移动、重命名
```
$ mv DIR1 DIR2
```
3.3	权限设置
16.	chmod
修改文件权限
```
$ chmod 777/755 file
$ chmod -R 600 folder
```
给组增加/收回写的权限
```
$ chmod g+w/g-w xxx
```
17.	chown
修改文件用户
```
$ chown root:root file
```
组
```
$ chgrp root a.txt
$ chown :root a.txt
```
18.	特殊目录符号
`.`
一个点，当前
`..`
两个点，上级
`~`
用户主目录，默认不打 cd 直接
`-`
上次目录
3.4	查找文件
19.	find
```
$ find PATH -name FILENAME
$ find /usr/bin -name scr*
$ find . -name “*.in” | xargs grep “ttttt”
```
20.	locate
用索引查找，速度很快，每天更新，可以用updatedb更新
```
$ updatedb
$ locate httpd.conf
```
3.5	文件夹大小
```
du -ah --max-depth=1
```
3.6	查看文件
21.	tail
```
$ tail -100 SystemOut.log
$ tail -f xxxxx
```
22.	cat
查看文件内容
```
$ cat file.txt
```
```
$ cat >> xxxx
```
追加

23.	head
查看文件头
```
$ head -n 100 sys.log
```
显示100行
24.	od
以特殊进制查看
```
$ od -t d /etc/my.cnf
```
3.7	文本编辑工具
25.	vi/vim
```
$ vi file_name
$ vim file_name
```
`hjkl`
左下上右移动光标

`Ctrl+f/Ctrl+b`
下一页/上一页

`u`
撤销编辑
`Ctrl+r`
撤销上次命令

`dd`
删除行
`3dd`
删除三行

`yy2`
`p`
拷贝多行、粘贴

`i`
进入编辑模式（进入编辑模式后才可自由编辑，按Esc退出）

`:q/:q!/:w/:wq/:x`
退出/强制退出/保存/保存退出/保存退出

`/xxx`
查找关键词

`n`
下一个匹配

`?xxxx`
逆序查找关键词
```
:set ff
:set ff=doc
set ff=unix
```
Windows文件换行符

26.	nano
简单的文本编辑工具
```
$ nano FILE
```
27.	emacs
强大的unix编辑工具

28.	less
分页查看（可翻页）
```
$ less FILENAME
$ ps -ef | less
```
`ctrl + F` - 向前移动一屏
`ctrl + B` - 向后移动一屏
`j` - 向前移动一行
`k` - 向后移动一行
`q` - 退出

29.	more
```
$ more SOMEFILE
```
4	系统设置
4.1	防火墙配置
30.	iptables (CentOS6)
```
$ service iptables stop/start
```
31.	firewall-cmd（CentOS7）
防火墙设置
```
# firewall-cmd --add-port=8080/tcp --permanent --zone=public
# firewall-cmd --reload
```
32.	ln
创建文件链接（软链接）
```
$ ln -s TARGET LINK_NAME
```
4.2	挂载/卸载
33.	mount
```
$ mount /dev/sdb1 newDisk
```
34.	unmount
用于外部设备
4.3	安装包
35.	yum (RHEL/CentOS)
安装包
```
# yum install
```
常用命令
```
# yum upgrade/update/search/install/remove/info installed/clean packages
```
组查询
```
# yum grouplist
```
组删除
```
# yum groupremove "MySQL Database"
```
安装软件组
```
# yum groupinstall "MATE Desktop"
```
查看已安装
```
# yum info installed
```
36.	rpm (RHEL/CentOS)
查询

```
# rpm -qa | grep mysql
```
安装rpm格式安装包

```
# rpm -ivh dejagnu-1.4.2-10.noarch.rpm
```
卸载

```
# rpm -e PACKAGE --nodeps
```


```
# rpm -e PACKAGE
```
删除（推荐yum remove/purge）
37.	apt-get (Debian/Ubuntu)
38.	dpkg (Debian)

```
# dpkg -i xxx.deb
```

4.4	网络设置

39.	ifconfig
```
$ ifconfig eth0
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
$ ifconfig eth0 down
$ ifconfig eth0 up
```

配置文件

```
# cat /etc/sysconfig/network-scripts
```

40.	设置hosts

```
# cat /etc/hosts
```
41.	DNS设置

```
# cat /etc/resolv.conf
```

42.	host
查询host的ip

```
$ host www.google.com
```

43.	traceroute
检测路由

```
$ traceroute www.google.com
```

4.5	执行周期任务
44.	cron

```
# service crond start
# service crond status
```


45.	crontab 
Linux的定时任务工具

```
$ crontab -l 
Crontab -e
```

Cron表达式：
基本格式 : 
`* * * * * command`
分 时 日 月 周 命令

配置记录
`/etc/crontab`

46.	zeus
第三方工具
宙斯zeus（替代方案）

4.6	磁盘信息
47.	df
显示磁盘使用情况，-h 可读格式

```
$ df -h
$ du -sh .
```

48.	du
查看目录信息

```
$ du -sk * | sort -rn
$ du -a | sort -rn
```

4.7	内存管理
49.	free
查看内存（总量、用量、空闲、swap (关注swap是否占用满)）

```
$ free -m
```
50.	ps
查看进程

```
$ ps -ef | grep java
```
|是管道符


```
$ ps aux
```
查看用户和进程
51.	kill
终止进程（9：直接杀掉）

```
$ kill -9 PID
```

通知终止，并产生core dump

```
$ kill -3 PID
```

重启

```
$ kill -1 PID
```

52.	pstree
树状结构显示运行中进程
53.	top
显示实时进程
大写P，按cpu使用率排序
大写M，按内存使用率排序
4.8	查看端口
54.	netstat

```
$ netstat -tunlp | grep 8080
$ netstat -tnl
$ netstat -apn
```

55.	lsof
查看打开文件相关进程

```
# lsof -i:8080
# lsof FILENAME
# lsof -c string
# lsof -u username
# lsof -i [protocol][@hostname][:service|port]
```

56.	bc
计算器

```
chkconfig --list
chkconfig --add xxx
chkconfig --level 345 xxx on
```

4.9	后台/屏幕管理
57.	screen
窗口管理器

```
$ sudo yum install screen
```

创建新窗口

```
$ screen
```

进入窗口，如果没有就创建新窗口

```
$ screen -R screen_name
```

退出窗口（后台继续）

```
Ctrl + a, d
```

结束窗口

```
$ exit
```
或

```
Ctrl + a, k
```

下一个/上一个窗口

```
Ctrl + a, n/Ctrl + a, p
```

进入窗口

```
$ screen -r screen_name/12345
```

列出所有窗口

```
$ screen -list
```

以session_name创建

```
$ screen -S session_name
```

重新进入

```
$ screen -r session_name
```
58.	Ctrl + z
当前任务放入后台
59.	&
后台执行

```
$ COMMAND > file.out &
```
60.	nohup

```
$ nohup COMMAND > file.out &
```
不退出
61.	jobs
查看后台的进程（包括Ctrl+z的进程）
62.	fg %1
后台进程调入前台
63.	bg %1
继续执行后台暂停的进程
64.	kill %1
杀死后台任务

4.10	系统服务
65.	systemctl（CentOS7）

```
# systemctl restat/start/stop/enable/disable/status firewalld/vncserver@:1.service
# systemctl status sshd.service
```

5	第三方工具
5.1	FTP
66.	scp

```
$ scp localfile root@ip:/home
```
Scp服务器传输协议
rz/lessrz

67.	ssh
远程连接

```
$ ssh ipaddress/hostname
$ ssh username@ipaddress
$ exit
```

68.	ftp

```
$ sudo yum install ftp
$ ftp hostname/ipaddress
cd/get/mget/put/mput/bye
bin（bineary mode）
```

69.	make
编译/安装

```
$ ./configration
$ make && make install
```

5.2	压缩/解压
70.	tar
-c 创建文件
-f 选择文件
-z gzip压缩算法
-v 在处理的同时观察文件
-x 解压
-C 目标目录
-t 列出压缩包内的文件

Examples:
```
tar -cf archive.tar foo bar  # Create archive.tar from files foo and bar.
  tar -tvf archive.tar         # List all files in archive.tar verbosely.
  tar -xf archive.tar          # Extract all files from archive.tar.
```

压缩、解压
```
$ tar -zcvf src.tar.gz src
$ tar -zxfv xxxxx xxxxxx.tar.gz -C folder_name
$ tar -czvf jh170215v403.tar.gz jh170215v403/* --exclude=jh170215v403/node_modules
```

XZ file

```
$ tar -Jxvf node-v6.10.0-linux-x64.tar.xz
```

查看压缩包内文件
```
$ tar tzvf apache-tomee-1.7.4-plus.tar.gz
$ tar jtvf src.tar.bz2
```

（-z，-j是压缩算法，可以不加）


71.	zip/unzip

```
$ unzip xxxx.zip -d xxxx
```
压缩和解压缩war、jar包

```
unzip tih.war -d tih
zip -r tih.war ./*
```

或者用jar命令

```
jar -cvfM0 tih.war tih
jar -xvf tih.war
```


压缩目录，排除某些子目录：

```
$ zip -r jhipster170206-v4.zip ./jhipster170206-v4 -x "./jhipster170206-v4/node_modules/*" -x "./jhipster170206-v4/target/*"
```

72.	gzip FILE
先tar包，再压缩gz
```
$ tar -cfv src.tar src
$ gzip src.tar
$ gzip -d FILE
```

5.3	下载工具
73.	wget
```
$ wget http://www.sss.com/zzz.zip
```


74.	curl

```
$ curl http://localhost:8080/index.html
```

TODO: -O 和 -o

75.	telnet

76.	lsof
恢复文件等
5.4	源码编译
77.	gcc
```
# gcc HelloWorld.c -o HelloWorld
# ./HelloWorld
```

环境变量保存到配置文件

```
echo “export PATH=$PATH:/root” >> /etc/rc.local
```

例子：编译安装Apache
```
# ./configure --prefix=/usr/lcoal/apache/ --enable-modules=most
# make && make install
```


6	其他
6.1	信息查询
78.	lscpu
查看CPU信息

79.	uptime
查看开机时间


```
vi /etc/sysctl.conf (too many open files)
```
80.	uname -a
显示操作系统内核版本信息

```
lsb_release -a
```
81.	date
查询、设置日期

```
# date +%Y%m%d
```
20170115
82.	设置时区
http://www.jb51.net/LINUXjishu/158824.html

```
$ tzselect
$ timeconfig
$ cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

83.	cal
日历

```
$ cal -y
```
84.	whoami
85.	which 
查找执行文件
86.	whereis
还能找man文件
87.	man COMMAND
查看命令的文档
f/b、空格，翻页 q 退出
搜索：/keyword

88.	info COMMAND
命令说明文档
f/b 翻页
6.2	管道符
89.	|
用于连接进程

90.	grep

```
$ grep ‘config’ config.txt
```

91.	sort

```
$ sort [-ntkr] filename
```
-n 数字排序
-t 指定分隔符
-k 第几列
-r 反序
92.	cpio
管道符cpio 文件备份

```
$ find /etc -name *.conf | cpio -cov > /tmp/conf.cpio
```
cpio 还原

```
$ cpio --absolute-filenames -icvu < /tmp/conf.cpio
```
93.	>
文件导向，输出至文件
```
$ ll > dir.out
$ tail -F > tail.out
```

6.3	其他的其他
94.	CTRL-c
Stop current command
95.	CTRL-z
Sleep program
96.	`sed` TODO: 
使用脚本处理文本，正则表达式
97.	echo
输出变量

```
$ echo $PATH
```
98.	clear
清屏
99.	touch file
创建文件

```
$ touch -t 0712250000 file (YYMMDDhhmm)
```
100.	ping IP/hostname
测试链接
6.4	设置环境变量
```
ln -s apache-maven-3.0.4 apache-maven
vi /etc/profile
         export M2_HOME=/usr/local/apache-maven
         PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
         export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE ...

source /etc/profile
```

