# Fedora 21 系统管理员指南 - 笔记

*System Administrator's Guide*
*- Deployment, Configuration, and Administration of Fedora 21*


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
yumdb info yum ＃查寻YUM数据库
yum install
yum groupinstall ＃同yum install ＠group
yum remove
yum history list
yum history summary
yum history info
``````

### 2.配置YUM

YUM配置文件: `/etc/yum.conf` 。分为 `[main]` 和 `[repository]` 两部分。
[repository]部分通常在 `/etc/yum.repos.d` 目录下通过独立的文件配置。

``````sh
＃__/etc/yum.conf__配置示例：
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
yum-config-manager --add-repo *repository_url* # 增加仓库
yum-config-manager --enable *repository* # 启用仓库
yum-config-manager --disable *repository*
``````

