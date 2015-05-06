# Fedora 21 系统管理员指南 - 笔记

*System Administrator's Guide*
*- Deployment, Configuration, and Administration of Fedora 21*


## 基本配置

### 1. locale

通过修改配置 `/etc/locale.conf` 或 `localectl` 命令。

``````bash
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

``````bash
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

``````bash
date # local time
date --utc
date +"%Y-%m-%d %H:%M:%S"
sudo date --set 11:22:33
sudo date --set 12:34:56 --utc
sudo date --set 2013-04-05 11:22:33
``````

传统操作RTC的命令，
`hwclock` 保存其设置在 `/etc/adjtime` 文件中：

``````bash
sudo hwclock
sudo hwclock --set --date "21 Oct 2014 21:17"
sudo hwclock --systohc # 用system time设置hardware clock
sudo hwclock --hctosys
``````

### 3. users & groups

__umask__ 在配置文件 `/etc/bashrc` 中，通常umask设定为 `022` .
组信息存储在 `/etc/group` 中。

管理用户、用户组的命令：

| 命令 ｜ 描述 ｜
|:-----|:-------|
| id | 显示用户及用户组ID |
| useradd, usermod, userdel | 增加、修改、删除用户的标准命令 |
| groupadd, groupmod, groupdel | 增加、修改删除用户组的标准命令|
| gpasswd | 管理 `/etc/group` 配置文件 |
| pwck, grpck | 检验密码、组等相关信息的命令  |
| pwconv, pwunconv | password与shadow password之间的转换 |
| grpconv, grpunconv | group shadow |

``````bash
sudo useradd zhangsan # 默认增加的用户是locked
passwd zhangsan # 指定密码并unlock用户
``````

__/etc/passwd__文件解释：

``````bash
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

``````bash
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

``````bash
ssbunny:x:1000:ssbunny
``````

1. 组名
2. 密码
3. GID
4. 组内用户，逗号分隔

`/etc/skel`　目录下的文件会在创建用户时复制到用户家目录，作为默认用户配置。

更改密码使用寿命：

``````bash
chage [options] username
``````
