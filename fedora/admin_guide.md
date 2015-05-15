# Fedora 21 系统管理员指南 - 笔记

*System Administrator's Guide*
*- Deployment, Configuration, and Administration of Fedora 21*

> * 1 - commands
> * 2 - system calls
> * 3 - library calls
> * 4 - special files
> * 5 - file formats and convertions
> * 6 - games for linux
> * 7 - macro packages and conventions
> * 8 - system management commands
> * 9 - others

## 基本配置

### 1. locale

通过修改配置 `/etc/locale.conf` 或 `localectl` 命令。

``````sh
localectl status
localectl list-locales | grep zh_
localectl set-locale LANG=en_US.utf8
``````

### 2. date & time

* `real-time clock` __RTC__ 硬件时钟，主板集成电路中，关机仍运行。
* `Universal Time` __UTC__ 系统时间
* `Locale Time` 带时区的本地时间

RTC通常采用UTC，也可以使用本地时间。

基于systemd的新命令：

``````sh
timedatectl  # 当前时间信息
sudo timedatectl set-time 11:22:33
sudo timedatectl set-local-rtc yes  # RTC时钟采用本地时间，设置为no采用UTC
sudo timedatectl set-time 2012-11-22 # 时间为00:00:00
sudo timedatectl set-time "2012-11-22 11:22:33"
timedatectl list-timezones
sudo timedatectl set-timezone Europe/Prague
sudo timedatectl set-ntp yes # 开启时间同步服务
``````

传统的date命令：

``````sh
date # local time
date --utc
date +"%Y-%m-%d %H:%M:%S"
sudo date --set 11:22:33
sudo date --set 12:34:56 --utc
sudo date --set 2013-04-05 11:22:33
``````

传统操作RTC的命令，
`hwclock` 保存其设置在 `/etc/adjtime` 文件中：

``````sh
sudo hwclock
sudo hwclock --set --date "21 Oct 2014 21:17"
sudo hwclock --systohc # 用system time设置hardware clock
sudo hwclock --hctosys
``````

### 3. users & groups

__umask__ 在配置文件 `/etc/bashrc` 中，通常umask设定为 `022` .
组信息存储在 `/etc/group` 中。

管理用户、用户组的命令：

| 命令                        | 描述                                |
|:--------------------------- |:----------------------------------- |
| id                          | 显示用户及用户组ID                  |
| useradd, usermod, userdel   | 增加、修改、删除用户的标准命令      |
| groupadd, groupmod, groupdel| 增加、修改删除用户组的标准命令      |
| gpasswd                     | 管理 `/etc/group` 配置文件          |
| pwck, grpck                 | 检验密码、组等相关信息的命令        |
| pwconv, pwunconv            | password与shadow password之间的转换 |
| grpconv, grpunconv          | group shadow                        |

``````bash

sudo useradd zhangsan # 默认增加的用户是locked
passwd zhangsan # 指定密码并unlock用户
``````

__/etc/passwd__文件解释：

``````sh
zhs:x:1000:1000:ZhangSan:/home/zhs:/bin/bash
``````

1. zhs: 用户名
2. x: 密码占位，指向shadow password.
3. 1000: UID, Fedora中，小于1000的是系统用户
4. 1000: GID, Fedora中，小于1000的是系统用户组
5. ZhangSan: GECOS, 用户详细信息
6. /home/zhs: 用户家目录
7. /bin/bash: 默认的shell

__/etc/shadow__文件解释：

``````sh
zhs:!!:14798:0:99999:7:::
``````

1. zhs: 用户名
2. !! 密码，锁定状态
3. 14798　上次更改日期，距离1970-01-01
4. 密码多少天后可以更改，０代表不限制
5. 密码多少天后失效，默认99999 代表永不失效
6. 7表示提醒用户密码将在７天后到期
7. 账号失效期限，密码到期时不修改，x天后账号被锁定
8. 账号到期日期，该日期后失效
9. 保留

__/etc/group__文件解释：

``````sh
ssbunny:x:1000:ssbunny
``````

1. 组名
2. 密码
3. GID
4. 组内用户，逗号分隔

`/etc/skel`　目录下的文件会在创建用户时复制到用户家目录，作为默认用户配置。

更改密码使用寿命：

``````sh
chage [options] username
``````

__自动登出__

使用户通过虚拟控制台或远程登录时，间隔一段时间不操作则自动登出。

1.安装 screen 包

``````sh
sudo yum install screen
``````

2.在 `/etc/profile` 开头加入

``````sh
trap "" 1 2 3 15
``````

3.在 `/etc/profie` 结尾加入

``````sh
SCREENEXEC="screen"
if [ -w $(tty) ]; then
  trap "exec $SCREENEXEC" 1 2 3 15
  echo -n 'Starting session in 10 seconds'
  sleep 10
  exec $SCREENEXEC
fi
``````

4.在 `/etc/screenrc` 结尾加入

``````sh
idle 120 quit
autodetach off
``````

或锁定屏幕：

``````sh
idle 120 lockscreen
autodetach off
``````

__创建组目录__

``````sh
mkdir /opt/myproject
groupadd myproject
chown root:myproject /opt/myproject
chmod 2775 /opt/myproject
usermod -aG myproject username
`````

## 包管理

### 1.YUM常用命令

``````sh
yum check-update # 检查哪些已安装包可更新
yum update mypackage # 更新指定包
yum update # 更新所有包
yum search term1 term2 # 按关键字查找包
yum list mypack* # 列出指定包
yum list all
yum list installed
yum list available
yum grouplist # 列出package groups
yum repolist # 列出仓库信息
yum info mypackage
yumdb info yum # 查寻YUM数据库
yum install
yum groupinstall # 同yum install ＠group
yum remove
yum history list
yum history summary
yum history info
``````

### 2.配置YUM

YUM配置文件: `/etc/yum.conf` 。分为 `[main]` 和 `[repository]` 两部分。
[repository]部分通常在 `/etc/yum.repos.d` 目录下通过独立的文件配置。

``````sh
＃/etc/yum.conf配置示例：
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=2
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=3
``````

__[main]__配置选项：

* `assumeyes`: **0**(默认)带提示信息，**1**不带提示信息，相当于yum -y
* `cachedir`: Yum缓存及数据库存放的绝对路径
* `debuglevel`: **1**到**10**级，越大调试信息越详细，默认为**2**，**0**表示禁用
* `exactarch`: **0**不包含系统架构信息；**1**(默认)包含架构信息，i686不会更新i386架构的包。
* `exclude`: 安装或更新时要排除的包列表
* `gpgcheck`: **0**关闭GPG签名；**1**(默认)开启GPG签名
* `groupremove_leaf_only`: **0**(默认)删除时**不**检查依赖；**1**删除时检查依赖
* `installonlypkgs`: 只安装不更新的包列表
* `installonly_limit`: installonlypkgs的列表上限，**0**禁用此特性，**3**默认
* `keepcache`:**0**(默认)不保存成功安装后的包；**1**保存
* `logfile`: yum的日志文件存放位置
* `multilib_policy`: **best**安装系统最匹配的包，如AMD64只安装64bit的包；**all**安装所有包
* `obsoletes`: **1**(默认)激活旧式进程逻辑；**0**禁用
* `plugins`: **0**全局禁用所有插件(不推荐)；**1**全局开启所有插件
* `reposdir`: 寻找仓库文件的目录，默认为/etc/yum.repos.d/
* `retries`: yum返回错误前的重试次数，**0**表示一直重试，默认为**10**

__[repository]__配置选项：

* `name`: 仓库名
* `baseurl`: url
* `enabled`: **0**禁用此仓库；**1**启用

YUM 变量：

* `$releasever`
* `$arch`
* `$basearch`
* `$YUM0-9`

增加自己的变量在 `/etc/yum/vars/` 下增加文件：

``````sh
echo "Fedora" > /etc/yum/vars/osname
``````

这样就可以在 **.repo** 文件中使用 **$osname** 变量

配置命令： **yum-config-manager**

``````sh
yum-config-manager main # 查看main配置
yum-config-manager --add-repo [repository_url] # 增加仓库
yum-config-manager --enable [repository] # 启用仓库
yum-config-manager --disable [repository]
``````

创建仓库： **createrepo**

``````sh
#所有需要的包放置在/mnt/local_repo/下
createrepo --database /mnt/local_repo
``````

### 3.Yum插件

``````sh
yum update --noplugins
yum update --disableplugin=plugin_name
yum install plugin_name
``````

## 基础架构服务

### 1.服务和守护进程

通过**systemctl**命令管理服务，Fedora 21中不推荐使用init脚本中古老的
**ntsysv**及**chkconfig**命令(使用/etc/rc.d/init.d)。

在boot时启用或停用服务：

``````sh
systemctl enable service_name.service
systemctl disable service_name.service
``````

检查服务状态：

``````sh
systemctl is-active service_name.service
systemctl status service_name.service
systemctl list-units --type=service # 列表显示
``````

列表字段含义：

* __UNIT__ systemd unit名称，这里是服务名
* __LOAD__ systemd unit是否加载
* __ACTIVE__ 高级别unit激活状态
* __SUB__ 低级别unit激活状态
* __JOB__ 该unit挂起的job
* __DESCRIPTION__ 描述信息

服务的启停：

``````sh
systemctl start service_name.service
systemctl stop service_name.service
systemctl restart service_name.service
``````

### 2.OpenSSH

有两类不同的配置：客户端程序(ssh,scp,sftp)和服务端(sshd)。
系统级的配置在 `/etc/ssh/` 目录下，用户级的配置在 `~/.ssh/` 下。

系统配置：

|  文件                           | 描述                                 |
|:--------------------------------|:-------------------------------------|
| /etc/ssh/moduli                 | 模数文件(ps: 很高端，不打算弄懂)     |
| /etc/ssh/ssh_config             | SSH客户端配置，会被~/.ssh/config覆盖 |
| /etc/ssh/sshd_config            | sshd进程的配置文件                   |
| /etc/ssh/ssh_host_ecdsa_key     | sshd进程使用的私有ECDSA key          |
| /etc/ssh/ssh_host_ecdsa_key.pub | sshd进程使用的公有ECDSA key          |
| /etc/ssh/ssh_host_key           | sshd进程版本1使用的RSA私有key        |
| /etc/ssh/ssh_host_key.pub       | sshd进程版本1使用的RSA公有key        |
| /etc/ssh/ssh_host_rsa_key       | sshd进程版本2使用的RSA私有key        |
| /etc/ssh/ssh_host_rsa_key.pub   | sshd进程版本2使用的RSA公有key        |
| /etc/pam.d/sshd                 | sshd进程使用的PAM配置文件            |
| /etc/sysconfig/sshd             | sshd服务的配置文件                   |

用户配置：

| 文件                   | 描述                 |
|:-----------------------|:---------------------|
| ~/.ssh/authorized_keys | 一组server认证公钥   |
| ~/.ssh/id_ecdsa        | user的ECDSA私有key   |
| ~/.ssh/id_ecdsa.pub    | ECDSA公有key         |
| ~/.ssh/id_rsa          | ssh版本2 RSA私有key  |
| ~/.ssh/id_rsa/pub      | ssh版本2 RSA公有key  |
| ~/.ssh/identity        | ssh版本1 RSA私有key  |
| ~/.ssh/identity.pub    | ssh版本1  RSA公有key |
| ~/.ssh/known_hosts     | server给的host key   |

__启动OpenSSH服务器__

``````sh
#管理脚本
systemctl start sshd.service
systemctl stop sshd.service
systemctl enable sshd.service
``````

启动SSH服务时，关闭其它不安全的连接：

``````sh
systemctl stop telnet.service
systemctl stop rsh.service
systemctl stop rlogin.service
systemctl stop vsftpd.service
``````

为了更安全起见，使用key pairs进行认证，需要先把密码认证关闭：
将 `/etc/ssh/sshd_config` 下的 `PasswordAuthentication` 设置为 `no`,
为每个用户生成独立的key.

注意：不要使用root用户生成，否则只有root用户能使用。

``````sh
ssh-keygen -t rsa #生成~/.ssh/id_rsa
ssh-copy-id -i ~/.ssh/id_rsa/pub user@hostname #copy到远程主机
``````

客户端登录：

``````sh
ssh username@hostname
``````

检查指纹的方法：

``````sh
ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
``````

从~/.ssh/known_hosts中删除host:

``````sh
ssh-keygen -R github.com
``````

不登录shell直接执行命令：

``````sh
ssh [username@]hostname command
``````

通过SSH远程传文件：

``````sh
scp localfile username@hostname:remotefile
scp ~/dirs/* username@hostname:~/dirs/ #传目录
scp username@hostname:remotefile localfile #从远程传至本地
``````

安全ftp:

``````sh
~]$ sftp username@hostname
sftp> ls [directory]
sftp> cd directory
sftp> mkdir directory
sftp> rmdir directory
sftp> put localfile [remotefile]
sftp> get remotefile [localfilen]
``````

X11转发：

``````sh
ssh -Y username@hostname
#之后可以通过shell启动X程序
``````

端口转发：

``````sh
ssh -L local-port:remote-hostname:remote-port username@hostname
#只要ssh到的host能访问的局域网主机，都有可能被转发至本地
``````

### 3.TigerVNC

用于提供图形化桌面访问。(ps:不怎么使用，略了)

``````sh
yum install tigervnc-server #服务端
yum install tigervnc #客户端浏览器，Windows也有相应工具
``````


## 服务器

### 1.Web服务器

(ps:天天搞Web，这部分略)

### 2.邮件服务器

Fedora使用**Dovecot**实现**POP**及**IMAP**。

``````sh
yum install dovecot
``````

(ps:这部分知识比较系统，用的时候再查，略)

### 3.目录服务器

__OpenLDAP__

术语：

* `entry` 条目，LDAP目录的独立单元，用唯一DN(Distinguished Name, 分辨名)标识。
* `attribute` 条目中的信息，例如entry表示公司，attribute则是其地址、传真等。
* `LDIF` LDAP Data Interchange Format,LDAP的数据交换格式，是一种文本型格式。

``````sh
#LDIF 示例
[id] dn: distinguished_name
attribute_type: attribute_value...
attribute_type: attribute_value...
...
``````

OpenLDAP基础包：

* openldap
* openldap-clients
* openldap-servers
* openldap-servers-sql

常用扩展包：

* nss-pam-ldapd
* mod-ldap

Server端命令：

* `slapacl` 属性列表
* `slapadd` 通过LDIF文件添加entry
* `slapauth` 认证
* `slapcat` 保证entry至LDIF文件
* `slapdn` DN列表
* `slapindex` 重建数据索引
* `slappasswd` 设置ldapmodify密码或slapd配置文件密码
* `slapschema` schema
* `slaptest` 检测LDAP服务配置

Client端命令：

* `ldapadd`
* `ldapcompare`
* `ldapdelete`
* `ldapexop`
* `ldapmodify`
* `ldapmodrdn`
* `ldappasswd`
* `ldapserch`
* `ldapurl`
* `ldapwhoami`

配置文件：

* **/etc/openldap/ldap.conf**
* **/etc/openldap/slapd.d/**

(ps:使用时参考OpenLDAP相关文档即可)

### 4.文件、打印服务器

__Samba__

Samba是SMB(Server Message Block)的开源实现，该协议的现代版本是CIFS(Common
Internat File System). 用于Windows、Linux、UNIX等系统间访问Windows文件或打印机

参考：
http://linux.vbird.org/linux_server/0370samba.php

__FTP__

Fedora提供两个FTP Server：

* **proftpd**
* **vsftpd**

vsftpd的配置文件：

* **/etc/rc.d/init.d/vsftpd** 初始化脚本
* **/etc/pam.d/vsftpd** PAM认证配置
* **/etc/vsftpd/vsftpd.conf** vsftpd的配置文件
* **/etc/vsftpd/ftpusers/** 不允许登录vsftpd的用户列表
* **/etc/vsftpd/user_list** 根据/etc/vsftpd/vsftpd.conf中的
`userlist_deny` 配置`YES`或`NO`，来设定允许或阻止登录的用户列表
* **/var/ftp/** 分享的文件，/var/ftp/pub/目录提供给匿名用户

__Printer__

(略)

以下时间相关的配置是系统管理员的高级技能，暂时不学习。

### 5.配置NTP

__使用chrony Suite__

__使用ntpd__

### 6.配置PTP

## 系统监控及自动化

### 1.系统监控工具

#### 1.1查看进程

__ps__

``````sh
ps ax
ps aux
ps ax | grep emacs
``````

__top__

``````sh
top
``````

字段含义：

1. **PID** the process ID
2. **USER** the effective username of the process owner
3. **PR** the priority
4. **NI** the nice value
5. **VIRT** the amount of virtual memory the process uses
6. **RES** the amount of non-swapped physical memory the process uses
7. **SHR** the amount of shared memory the process uses
8. **%CPU** the percentage of the CPU usage
9. **%MEM** the percentage of the memory usage
10. **TIME+** the cumulated CPU time
11. **COMMAND** the name of the executable file

交互命令：

| 命令         | 描述           |
|:-------------|:---------------|
| Enter, Space | 立即刷新       |
| h, ?         | 帮助文档       |
| k            | 杀进程         |
| n            | 显示的进程数量 |
| u            | 按用户排序     |
| M            | 按内存排序     |
| P            | 按CPU排序      |
| q            | 退出           |

#### 1.2查看内存

``````sh
free
free -m
``````

#### 1.3查看CPU

``````sh
lscpu
``````

#### 1.4块设备和文件系统

查看块设备：

``````sh
lsblk #树结构
lsblk -l #列表结构
sudo blkid
sudo blkid /dev/vda1
sudo blkid -po udev device_name
``````

磁盘分区：

``````sh
partx -s device_name
``````

查看挂载的文件系统：

``````sh
findmnt
findmnt -l
findmnt -t ext4
``````

磁盘空间使用情况：

``````sh
df
df -h
``````

目录中的文件使用情况：

``````sh
du ~/
du -h /etc #可读的大小
du -sh ~/code #汇总目录大小
``````

#### 1.5硬件信息

查看PCI设备：

``````sh
lspci
lspci -v
lspci -vv
``````

查看USB：

``````sh
lsusb
lsusb -v
``````

查看PCMCIA设备：

``````sh
lspcmcia
lspcmcia -v
``````

#### 1.6使用Net-SNMP监控性能

SNMP Deamon: *net-snmp*

``````sh
systemctl start snmpd.service
systemctl enable snmpd.service
systemctl stop snmpd.service
systemctl disable snmpd.service
systemctl restart snmpd.service
systemctl reload snmpd.service
``````

(ps: 这部分比较系统，更多内容可参考Net-SNMP官网)

### 2.管理日志

#### 2.1.查找日志文件

大多数日志放在 `/var/log/` 目录，httpd或samba等应用会在其下创建独立的目录存放。
**logrotate**工具可用来转储日志，如按天拆分、按大小、发Email等。

#### 2.2.配置Rsyslog

**rsyslog**的配置文件是 `/etc/rsyslog.conf` .

#### 2.3.Rsyslog队列

#### 2.4.Rsyslog模块

#### 2.5.Rsyslog与Journal交互

#### 2.6.结构化Rsyslog日志

#### 2.7.调试Rsyslog

#### 2.8.使用Journal

#### 2.9.图形化日志管理

### 3.自动化任务

#### 3.1Cron & Anacron

#### 3.2At & Batch

### 4.OProfile



## 内核、模块和驱动配置

### 1.GRUB 2

**配置文件**位置：

* BIOS：**/boot/grub2/grub.cfg**
* UEFI：**/boot/efi/EFI/redhat/grub.cfg**

配置文件不直接修改，而是通过 `/usr/sbin/grub2-mkconfig` 命令修改。
命令依据**/boot/**中的Linux内核生成配置文件。

模板文件在： **/etc/grub.d/** 中，自定义设置在 **/etc/default/grub** 文件中。
每次更新/etc/default/grub文件时要重新生成 `grub.cfg` 文件。如：

``````sh
grub2-mkconfig -o /boot/grub2/grub.cfg
``````

配置文件中每个 `menuentry` 代表一个GRUB2引导菜单入口，后面跟着一个title，
一组选项和一个花括号；花括号内的内容要缩进，其中 `linux16` 指向Linux内核
(在UEFI系统中是 `linuxefi` )。`initrd16` 指向RAM镜像。如果**/boot**是独立分区，
则是基于/boot的相对路径。

**定义GRUB2菜单**

修改 `/etc/default/grub` 中的 `GRUB_DEFAULT` 设置默认入口：

1. 设置为 `saved`
2. 设置为具体入口名

值为saved时，会根据**/boot/grub2/grubenv**中的**saved_entry**确定入口，
此时可以用 `grub2-set-default` 命令来进行设置，此命令也可以接数字：

``````sh
grub2-set-default 2
``````

值为具体名时，可以根据以下命令找到可用的入口：

``````sh
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
``````

一次性的**编辑GRUB**可以在引导菜单出现时选中对应菜单，按 `e` 键。
永久改变需要改/etc/default/grub文件，如采用*emergency mode*启动：

``````sh
GRUB_CMDLINE_LINUX="emergency"
``````

执行**grub2-mkconfig**命令时，GRUB2通过**/etc/grub.d/**目录查找：

* `/etc/grub.d/10_linux` 查找同分区下安装的Linux内核
* `/etc/grub.d/30_os-prober` 查找其它的操作系统

`40_custom` 则用来自定义入口，它可以被编辑或复制。

**自定义菜单:**

> $$ 优先备份好 /etc/grub.d/ 目录，以做还原。

1. 拷贝 /boot/grub2/grub.cfg (或 /boot/efi/EFI/redhat/grub.cfg )的内容至
/etc/grub.d/40_custom 文件底部；
2. 删除 menuentry 部分之外的其它脚本代码；
3. 删除(或取消执行权) /etc/grub.d/ 目录下的脚本，只保留 00_header,
40_custom, 01_users(如果存在), README 即可；
4. 按需要修改、添加、删除 40_custom 中的入口菜单；
5. 执行 grub2-mkconfig -o /boot/grub2/grub.cfg



### 2.更新内核

### 3.内核模块
