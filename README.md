## 【阿里云镜像】基于YUM方式搭建Zabbix监控平台

### 一、参考链接

[阿里巴巴开源镜像站-OPSX镜像站-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/mirror/?spm=a2c6h.13651102.0.0.78c31b11Yqe2kC&serviceType=mirror)

[zabbix镜像-zabbix下载地址-zabbix安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/zabbix?spm=a2c6h.13651102.0.0.74431b11IkZ596)

### 二、Zabbix简介

> Zabbix 是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。它能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位/解决存在的各种问题。

**下载地址**：https://mirrors.aliyun.com/zabbix/

### 三、Zabbix监控平台安装步骤

### 1、登录centos系统

```sh
C:\Users\xybdiy>ssh root@192.168.200.50
root@192.168.200.50's password:
Last login: Fri Dec 24 22:31:27 2021
[root@centos ~]# hostnamectl
   Static hostname: centos
         Icon name: computer-vm
           Chassis: vm
        Machine ID: f6fc8fb7991c4c518238af7c75f16046
           Boot ID: daae3878cb56458bac0b89acff8aa851
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-1160.el7.x86_64
      Architecture: x86-64
[root@centos ~]#
```

### 2、关闭防火墙和SELINUX安全模式

```sh
# 关闭防火墙
[root@centos ~]# systemctl stop firewalld
[root@centos ~]# systemctl disable firewalld


# 关闭SELINUX安全模式（重启生效）
[root@centos ~]# setenforce 0
[root@centos ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

[root@centos ~]#reboot
[root@centos ~]# getenforce
Disabled
```

### 3、更新YUM源为阿里云镜像源

```sh
[root@centos ~]# yum clean all
Loaded plugins: fastestmirror
Repodata is over 2 weeks old. Install yum-cron? Or run: yum makecache fast
Cleaning repos: base extras updates
Cleaning up list of fastest mirrors
[root@centos ~]# yum makecache
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.aliyun.com
base                                        | 3.6 kB  00:00:00
extras                                      | 2.9 kB  00:00:00
updates                                     | 2.9 kB  00:00:00
(1/10): base/7/x86_64/group_gz              | 153 kB  00:00:00
(2/10): base/7/x86_64/filelists_db          | 7.2 MB  00:00:01
(3/10): extras/7/x86_64/filelists_db        | 259 kB  00:00:00
(4/10): extras/7/x86_64/primary_db          | 243 kB  00:00:00
(5/10): extras/7/x86_64/other_db            | 145 kB  00:00:00
(6/10): base/7/x86_64/other_db              | 2.6 MB  00:00:00
(7/10): base/7/x86_64/primary_db            | 6.1 MB  00:00:02
(8/10): updates/7/x86_64/filelists_db       | 7.0 MB  00:00:01
(9/10): updates/7/x86_64/other_db           | 903 kB  00:00:00
(10/10): updates/7/x86_64/primary_db        |  13 MB  00:00:02
Metadata Cache Created
[root@centos ~]#
```

### 4、安装LNMP环境

```sh
[root@centos ~]# yum install httpd httpd-devel mariadb mariadb-server mariadb-devel php-common php-gd php-mbstring php-xml php-bcmath php-mysql php-cli php-devel php-pear -y
```


### 5、添加Zabbix扩展源

```sh
https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
```

```sh
[root@centos ~]# rpm -ivh https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
Retrieving https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-2.el7.noarch.rpm
warning: /var/tmp/rpm-tmp.Xp3vAb: Header V4 RSA/SHA512 Signature, key ID a14fe591: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:zabbix-release-4.0-2.el7         ################################# [100%]
[root@centos ~]#
```

![image-20211223224002411](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112232240664.png)

### 6、更新Zabbix.repo源

```sh
#修改/etc/yum.repos.d/zabbix.repo内容如下：
cat>/etc/yum.repos.d/zabbix.repo<<EOF
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX-A14FE591
[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX
gpgcheck=1
EOF
```

![image-20211224094321465](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112240943417.png)

```sh
[root@centos ~]# cat>/etc/yum.repos.d/zabbix.repo<<EOF
> [zabbix]
> name=Zabbix Official Repository - $basearch
> baseurl=https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/$basearch/
> enabled=1
> gpgcheck=1
> gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX-A14FE591
> [zabbix-non-supported]
> name=Zabbix Official Repository non-supported - $basearch
> baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7/$basearch/
> enabled=1
> gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX
> gpgcheck=1
> EOF
[root@centos ~]#
[root@centos ~]# cat /etc/yum.repos.d/zabbix.repo
[zabbix]
name=Zabbix Official Repository -
baseurl=https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7//
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX-A14FE591
[zabbix-non-supported]
name=Zabbix Official Repository non-supported -
baseurl=https://mirrors.aliyun.com/zabbix/non-supported/rhel/7//
enabled=1
gpgkey=https://mirrors.aliyun.com/zabbix/RPM-GPG-KEY-ZABBIX
gpgcheck=1
[root@centos ~]#
```

### 7、安装Zabbix相关软件包

```sh
[root@centos ~]# yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent -y
[root@centos ~]# sed -i '/date.timezone/i date.timezone = PRC' /etc/php.ini
```


### 8、启动相关服务

```sh
[root@centos ~]# systemctl start httpd
[root@centos ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@centos ~]# systemctl start mariadb
[root@centos ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@centos ~]#
```

### 9、创建数据库&密码授权

```sh
[root@centos ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> grant all on zabbix.* to zabbix@'%' identified by 'zabbix';

MariaDB [(none)]> grant all on zabbix.* to zabbix@localhost identified by 'zabbix';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> Ctrl-C -- exit!
Aborted
```

### 10、导入基础数据库

```sh
[root@centos ~]# vim /usr/share/doc/zabbix-server-mysql-4.0.37/create.sql.gz
[root@centos ~]# zcat /usr/share/doc/zabbix-server-mysql-4.0.37/create.sql.gz|mysql -uzabbix -p123456 zabbix
[root@centos ~]#
```

![image-20211224095449077](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112240954752.png)

```sh
[root@centos ~]# rpm -qa | grep zabbix
zabbix-release-4.0-2.el7.noarch
zabbix-web-mysql-4.0.37-1.el7.noarch
zabbix-web-4.0.37-1.el7.noarch
zabbix-agent-4.0.37-1.el7.x86_64
zabbix-server-mysql-4.0.37-1.el7.x86_64
[root@centos ~]#
```

### 11、设置时区

```sh
[root@zabbix-server ~]# vim /etc/php.ini
date.timezone = PRC
[root@zabbix-server ~]# vim /etc/httpd/conf.d/zabbix.conf
```

![image-20211224103033489](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241030640.png)

### 12、修改Zabbix配置文件

```sh
[root@zabbix-server ~]# vim /etc/zabbix/zabbix_server.con
[root@zabbix-server ~]# grep -n '^'[a-Z] /etc/zabbix/zabbix_server.conf
38:LogFile=/var/log/zabbix/zabbix_server.log
49:LogFileSize=0
72:PidFile=/var/run/zabbix/zabbix_server.pid
82:SocketDir=/var/run/zabbix
91:DBHost=localhost
100:DBName=zabbix
116:DBUser=zabbix
124:DBPassword=zabbix
132:DBSocket=/var/lib/mysql/mysql.sock
357:SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
472:Timeout=4
515:AlertScriptsPath=/usr/lib/zabbix/alertscripts
526:ExternalScripts=/usr/lib/zabbix/externalscripts
562:LogSlowQueries=3000
[root@zabbix-server ~]#
```

### 13、启动Zabbix

```sh
[root@zabbix-server ~]# systemctl start zabbix-server
[root@zabbix-server ~]# systemctl enable zabbix-server
Created symlink from /etc/systemd/system/multi-user.target.wants/zabbix-server.service to /usr/lib/systemd/system/zabbix-server.service.
[root@zabbix-server ~]#
[root@zabbix-server ~]# netstat -ntpl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      2088/mysqld
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      927/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1040/master
tcp        0      0 0.0.0.0:10051           0.0.0.0:*               LISTEN      2350/zabbix_server
tcp6       0      0 :::80                   :::*                    LISTEN      2305/httpd
tcp6       0      0 :::22                   :::*                    LISTEN      927/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1040/master
tcp6       0      0 :::10051                :::*                    LISTEN      2350/zabbix_server
[root@zabbix-server ~]#
```

### 14、Zabbix WEB GUI安装配置

通过浏览器Zabbix_WEB验证，通过浏览器访问http://192.168.200.50/zabbix

![image-20211224102723278](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241027830.png)

显示PHP版本信息等内容

![image-20211224103214993](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241032306.png)

填写连接数据库的必要信息

![image-20211224103331550](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241033511.png)

填写Zabbix服务端的详细信息

![image-20211224103438488](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241034424.png)

确认配置信息

![image-20211224103524297](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241035962.png)

安装Zabbix

![image-20211224103614575](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241036551.png)

输入账号登录

![image-20211224103658085](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241036289.png)

![image-20211224103727639](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241037743.png)

设置中文界面

![image-20211224103835997](https://gitee.com/xuyebaodiy/img/raw/master/%E9%98%BF%E9%87%8C%E4%BA%91%E9%95%9C%E5%83%8F%E9%85%8D%E7%BD%AE%E6%88%AA%E5%9B%BE/202112241038612.png)

**至此，Zabbix平台搭建完成。**

