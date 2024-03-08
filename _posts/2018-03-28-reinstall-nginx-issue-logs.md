---
title: 记一次重装nginx时遇到的问题
title_en: reinstall-nginx-issue-logs
date: 2018-03-28 00:25:36
tags:
  - Log
  - Nginx
  - Ubuntu
---

## 起因

之前在网上看文章提到说，通过 apt-get 的方式安装 nginx，可能安装不是最新版本的情况，考虑到在 Ubuntu 下第一次安装 nginx，以后肯定会有卸载重新安装新版本的需求，刚好刚开始学习 nginx，索性练习下卸载重装的过程。

---

## 安装

使用 Ubuntu 下的包管理工具 apt 来安装 nginx

```
$ sudo apt-get install nginx
```

## 卸载

同样使用 apt 来卸载 nginx

```
$ sudo apt-get remove nginx
```

`sudo apt-get autoremove`命令帮我们卸载不再需要的依赖包
在这里我们卸载 nginx 相关的依赖包

```
$ sudo apt-get autoremove
```

删除 nginx 配置文件夹

```
$ sudo rm -rf /etc/nginx
```

## 重新安装 nginx

```
# 更新源
$ sudo apt update
$ sudo apt-get install nginx
```

然而控制台报错：

<!-- more -->

```
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
invoke-rc.d: initscript nginx, action "start" failed.
dpkg: error processing package nginx-core (--configure):
 subprocess installed post-installation script returned error exit status 1
dpkg: dependency problems prevent configuration of nginx:
 nginx depends on nginx-core (>= 1.10.3-0ubuntu0.16.04.2) | nginx-full (>= 1.10.3-0ubuntu0.16.04.2) | nginx-light (>= 1.10.3-0ubuntu0.16.04.2) | nginx-extras (>= 1.10.3-0ubuntu0.16.04.2); however:
  Package nginx-core is not configured yet.
  Package nginx-full is not installed.
  Package nginx-light is not installed.
  Package nginx-extras is not installed.
 nginx depends on nginx-core (<< 1.10.3-0ubuntu0.16.04.2.1~) | nginx-full (<< 1.10.3-0ubuntu0.16.04.2.1~) | nginx-light (<< 1.10.3-0ubuntu0.16.04.2.1~) | nginx-extras (<< 1.10.3-0ubuntu0.16.04.2.1~); however:
  Package nginx-core is not configured yet.
 No apport report written because the error message indicates its a followup error from a previous failure.
 Package nginx-full is not installed.
  Package nginx-light is not installed.
  Package nginx-extras is not installed.

dpkg: error processing package nginx (--configure):
 dependency problems - leaving unconfigured
Processing triggers for libc-bin (2.23-0ubuntu9) ...
Errors were encountered while processing:
 nginx-core
 nginx
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

查看`nginx.service`状态

```
$ systemctl status nginx.service
```

```
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: failed (Result: exit-code) since Sat 2018-03-17 23:16:40 CST; 1min 18s ago
  Process: 26795 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=1/FAILURE)
 Main PID: 18259 (code=exited, status=0/SUCCESS)

Mar 17 23:16:40 VM-0-5-ubuntu systemd[1]: Starting A high performance web server and a reverse proxy server...
Mar 17 23:16:40 VM-0-5-ubuntu nginx[26795]: nginx: [emerg] open() "/etc/nginx/nginx.conf" failed (2: No such file or direMar 17 23:16:40 VM-0-5-ubuntu nginx[26795]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 17 23:16:40 VM-0-5-ubuntu systemd[1]: nginx.service: Control process exited, code=exited status=1
Mar 17 23:16:40 VM-0-5-ubuntu systemd[1]: Failed to start A high performance web server and a reverse proxy server.
Mar 17 23:16:40 VM-0-5-ubuntu systemd[1]: nginx.service: Unit entered failed state.
Mar 17 23:16:40 VM-0-5-ubuntu systemd[1]: nginx.service: Failed with result 'exit-code'.
```

仔细看报错信息，获取到两个信息点：

1. Package nginx-core is not configured yet.
2. Package nginx-full/nginx-light/nginx-extras is not installed.

于是上网搜，网上找说可能是`apache`占用了`80`端口导致报错

查看端口占用情况

```
$ sudo netstat -nlp
```

```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1287/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           891/dhclient
udp        0      0 172.21.0.5:123          0.0.0.0:*                           1231/ntpd
udp        0      0 127.0.0.1:123           0.0.0.0:*                           1231/ntpd
udp        0      0 0.0.0.0:123             0.0.0.0:*                           1231/ntpd
udp6       0      0 :::123                  :::*                                1231/ntpd
```

发现 80 端口没被占用，而且`systemctl status nginx.service`的提示信息也没提及 80 端口被占用，如果提示信息中说 80 端口被占用，可以执行以下两步试试：

终止 apache 运行

```
$ sudo service apache2 stop
```

重新安装 nginx

```
$ sudo apt-get install nginx
```

> 了解详细信息请看[askubuntu 上的提问][1]

---

上述情况与我的并不相符，于是换个思路，报错信息中提及 nginx 的配置文件不存在，思考为什么卸载重装 nginx，却没有生成配置文件？
和第一次安装的时候不一样，然后百度上搜`nginx卸载重装后配置文件没有重新生成`，找到了类似的问题，给出了如下的操纵步骤：

卸载 nginx 不保留配置文件

```
$ sudo apt-get --purge remove nginx

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libtiff5 libvpx3 libxpm4
  libxslt1.1 nginx-common nginx-core
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  nginx*
0 upgraded, 0 newly installed, 1 to remove and 205 not upgraded.
2 not fully installed or removed.
After this operation, 37.9 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 66386 files and directories currently installed.)
Removing nginx (1.10.3-0ubuntu0.16.04.2) ...
Setting up nginx-core (1.10.3-0ubuntu0.16.04.2) ...
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
invoke-rc.d: initscript nginx, action "start" failed.
dpkg: error processing package nginx-core (--configure):
 subprocess installed post-installation script returned error exit status 1
Errors were encountered while processing:
 nginx-core
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

卸载自动安装且不再需要的依赖包

```
$ sudo apt-get autoremove

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  fontconfig-config fonts-dejavu-core libfontconfig1 libgd3 libjbig0 libjpeg-turbo8 libjpeg8 libtiff5 libvpx3 libxpm4
  libxslt1.1 nginx-common nginx-core
0 upgraded, 0 newly installed, 13 to remove and 205 not upgraded.
1 not fully installed or removed.
After this operation, 9,745 kB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 66383 files and directories currently installed.)
Removing nginx-core (1.10.3-0ubuntu0.16.04.2) ...
Removing libgd3:amd64 (2.1.1-4ubuntu0.16.04.8) ...
Removing libfontconfig1:amd64 (2.11.94-0ubuntu1.1) ...
Removing fontconfig-config (2.11.94-0ubuntu1.1) ...
Removing fonts-dejavu-core (2.35-1) ...
Removing libtiff5:amd64 (4.0.6-1ubuntu0.2) ...
Removing libjbig0:amd64 (2.1-3.1) ...
Removing libjpeg8:amd64 (8c-2ubuntu8) ...
Removing libjpeg-turbo8:amd64 (1.4.2-0ubuntu3) ...
Removing libvpx3:amd64 (1.5.0-2ubuntu1) ...
Removing libxpm4:amd64 (1:3.5.11-1ubuntu0.16.04.1) ...
Removing libxslt1.1:amd64 (1.1.28-2.1ubuntu0.1) ...
Removing nginx-common (1.10.3-0ubuntu0.16.04.2) ...
Processing triggers for libc-bin (2.23-0ubuntu9) ...
Processing triggers for man-db (2.7.5-1) ...
```

筛选已安装软件包中与 nginx 有关的

```
$ dpkg --get-selections | grep nginx

nginx-common                                    deinstall
```

卸载`nginx-common`不保留配置文件

```
$ sudo apt-get --purge remove nginx-common

Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages will be REMOVED:
  nginx-common*
0 upgraded, 0 newly installed, 1 to remove and 205 not upgraded.
After this operation, 0 B of additional disk space will be used.
Do you want to continue? [Y/n] y
(Reading database ... 66243 files and directories currently installed.)
Removing nginx-common (1.10.3-0ubuntu0.16.04.2) ...
Purging configuration files for nginx-common (1.10.3-0ubuntu0.16.04.2) ...
dpkg: warning: while removing nginx-common, directory '/var/www/html' not empty so not removed
```

重新安装 nginx

```
$ sudo apt-get install nginx
...
```

查看版本号，执行 nginx 配置文件语法检测

```
$ nginx -v
nginx version: nginx/1.10.3 (Ubuntu)

$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

### 虽然 nginx 已经重装好了，但是一路下来还有几个困惑：

1. apt-get autoremove 到底是按什么规则卸载软件？
2. 为什么 nginx-common 没有在一开始的时候卸载？

#### 1. apt-get autoremove 到底是按什么规则卸载软件？

`apt-get autoremove`：删除，自动安装的，且不再需要的
（不被其他软件当作依赖的）软件包

> 举个栗子: 通过 apt-get 方式安装 nginx 时，会通过引导自动安装所需的依赖包（nginx-core），而当我们卸载 nginx 后，
> 那些自动安装的依赖包就成为不再需要的软件包（nginx-core），通过`apt-get autoremove`会自动清理它们

既然会自动删除不再需要的软件包，那么为什么 nginx-common 没被删除？

#### 2. 为什么 nginx-common 没有在`apt-get autoremove`的时候卸载？

不知道细心的童鞋发现没，我们在卸载 nginx 的时候，
手动删除了 nginx 的配置文件夹（`sudo rm -rf /etc/nginx`）。
那这些配置文件夹和`nginx-common`软件包有什么关系呢？

通过查找 ubuntu packages，找到了[Ubuntu16.0.4 下 nginx-common 软件包的文件清单][2]。

按照清单上的目录手动排查，发现本机上除了手动删除的`/etc/nginx`目录不存在，文件清单内的其他文件都没有被删除，
<del>猜测可能是 nginx-common 软件包完整性被破坏，导致`autoremove`的时候没有被删除，当然这只我的猜测。</del>

##### 问题回答后续补充:

整理完全卸载 nginx 命令时，发现执行如下命令，ngnix 的配置文件并没有被删除

```
$ sudo apt-get remove --purge nginx
```

查阅 ubuntu packages 发现，nginx 依赖 nginx-core，nginx-core 依赖 nginx-common，且其中 nginx 的配置文件属于 nginx-common 软件包配置文件的一部分。

执行如下命令进一步验证

```
# 卸载nginx-common ，nginx配置文件不会被删除
$ sudo apt-get remove nginx-common
...
# 卸载nginx-common，nginx配置文件已经被删除（包括但不限于）
$ sudo apt-get remove --purge nginx-common
$ find /etc/nginx/
find: ‘/etc/nginx/’: No such file or directory
```

##### 推翻之前的猜测：

`apt-get autoremove`执行时，`nginx-common`软件包已经被选择用于卸载( `deinstall` )，但是实际还没有卸载。

```
$ dpkg --get-selections| grep nginx
nginx-common                                    deinstall
```

[了解更多关于 deinstall，点击这里][6]，如果有知道为什么的同学请不吝赐教。

---

## 命令汇总

```bash
# 完全卸载nginx
$ sudo apt-get remove --purge nginx
$ sudo apt-get autoremove --purge

# 更新nginx，保留配置文件
# 亲测nginx.conf不会被删除和覆盖，但保险起见还是建议先备份
$ sudo apt-get remove nginx
$ sudo apt-get autoremove
$ sudo apt update
$ sudo apt-get install nginx

# 安装软件包
# sudo apt-get install 软件包名称`
# eg:
$ sudo apt-get install nginx

# 卸载软件包
# sudo apt-get remove 软件包名称
# eg:
$ sudo apt-get remove nginx

# 卸载软件包且不保留配置文件
# sudo apt-get remove --purge 软件包名称
# eg:
$ sudo apt-get remove --purge nginx

# 卸载自动安装的软件包且不保留配置文件
# sudo apt-get autoremove --purge

# 列出本地软件包列表
# dpkg --get-selections [| grep 筛选关键字]
# eg:（列出本地所有软件包）
$ dpkg --get-selections
# eg:（列出本地与ngnix有关的软件包）
$ dpkg --get-selections | grep nginx

# 查看进程信息
# ps -ef [| grep 筛选关键字]
# eg:（列出所有进程信息）
$ ps -ef
# eg:（列出与nginx有关的进程信息）
$ ps -ef | grep nginx

# 查看端口占用信息
$ sudo netstat -nlp

```

---

## 参考链接汇总

> [Package nginx-core not configured yet.Sub-process :/user/bin/dpkg returned an error code(1)][5]

> [nginx fail to install on Ubuntu 15.10 server][1]

> [Ubuntu 14.04 上卸载 nginx 之后重新安装没有重新生成配置文件的解决方法][3]

> [在 xenial 发行版中 all 硬件架构下的 nginx-common 软件包文件清单][2]

> [What is difference between the options “autoclean”, “autoremove” and “clean”?][4]

> [dpkg --get-selections shows packages marked “deinstall”][6]

[1]: https://askubuntu.com/questions/739939/nginx-fail-to-install-on-ubuntu-15-10-server
[2]: https://packages.ubuntu.com/xenial/all/nginx-common/filelist
[3]: https://www.mobibrw.com/2017/6059/comment-page-1
[4]: https://askubuntu.com/questions/3167/what-is-difference-between-the-options-autoclean-autoremove-and-clean
[5]: https://stackoverflow.com/questions/40341889/package-nginx-core-not-configured-yet-sub-process-user-bin-dpkg-returned-an-er
[6]: https://askubuntu.com/questions/165951/dpkg-get-selections-shows-packages-marked-deinstall
