# Fedora 21 系统管理员手册 - 笔记

*System Administrator's Guide*
*- Deployment, Configuration, and Administration of Fedora 21*


## 基本配置

### 1.locale

通过修改 `/etc/locale.conf` 配置或 `localectl` 命令。

``````bash
localectl status
localectl list-locales | grep zh_
localectl set-locale LANG=en_US.utf8
``````

### 2.date & time

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

### users & groups
