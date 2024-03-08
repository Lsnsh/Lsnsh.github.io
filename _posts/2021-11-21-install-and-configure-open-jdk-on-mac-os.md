---
title: 在 MacOS 上安装并配置 OpenJDK
title_en: install-and-configure-open-jdk-on-mac-os
date: 2021-11-21 12:01:29
tags:
  - Java
  - MacOS
---

## 背景

陆续升级到 `MacOS BigSur/Monterey` 后，电脑上装了两个虚拟机（`PD、VMWare Funison`），日常开发体验不好。一番取舍后，备份好数据，抹除硬盘给 `MacBook` 装了双系统，然后要重新装环境。

## 安装 OpenJDK

[OpenJDK][13] 是 `Java` 的开源版本，有 `Oracle` 构建维护的，也有[其他组织或公司构建](#其他-openjdk)并维护的，这里选择安装 `Oracle` 构建的（`OpenJDK builds from Oracle`）。

> Since September 2017, Oracle provides JDK releases under a free open source license (similar to that of Linux). Availability and community support of OpenJDK releases provided by Oracle is listed separately on jdk.java.net.
>
> 自 2017 年 9 月起，Oracle 在免费开源许可（类似于 Linux）下提供 JDK 版本。 Oracle 提供的 OpenJDK 版本的可用性和社区支持在 jdk.java.net 上单独列出。—— [Oracle Java SE Support Roadmap][3]

<!-- more -->

### 安装 Homebrew

使用 [Homebrew][0] 安装 `OpenJDK`，方便管理（查看、更新、卸载）

在终端中执行安装脚本：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 安装 OpenJDK@8

使用 `brew` 命令行工具：

```bash
brew install openjdk@8
```

### 配置 OpenJDK@8

根据输出的安装信息的提示，为了让 `Java wrappers` 找到 `JDK`，需要手动建立链接：

```bash
# For the system Java wrappers to find this JDK, symlink it with
sudo ln -sfn /usr/local/opt/openjdk@8/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-8.jdk
```

配置 `$JAVA_HOME` 环境变量，找到 `.bash_profile/.bashrc/.zshrc` 等配置文件中的任意一个，添加下面这行代码：

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v1.8)
```

然后退出终端重新打开，或者重新加载配置文件：

```bash
# 以 .zshrc 为例
source ~/.zshrc
```

在终端中使用 `java` 命令行工具，来检查配置是否生效：

```bash
java -version
```

输出版本信息，表示配置成功：

```bash
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-bre_2021_10_20_23_15-b00)
OpenJDK 64-Bit Server VM (build 25.312-b00, mixed mode)
```

## JDK 版本切换（会话级别）

新项目应该都在用 `JDK@11` 或者 `JDK@17`，但行业内目前依然有不少项目在使用 `JDK@8`。

仅供学习参考，实际使用可以看看 [jenv - Manage your Java environment][6]

### 动态查找 Java Home

![](/assets/images/10-在MacOS上安装并配置OpenJDK/developer.apple.com_library_archive_qa_qa1170__index.html.png)

使用 `/usr/libexec/java_home` 命令行工具（支持动态查找 `Java Home`，默认为最新版本 `JDK` 的 `Java Home`）

```bash
➜  ~ /usr/libexec/java_home
/usr/local/Cellar/openjdk/17.0.1/libexec/openjdk.jdk/Contents/Home

➜  ~ /usr/libexec/java_home -v1.8
/usr/local/Cellar/openjdk@8/1.8.0+312/libexec/openjdk.jdk/Contents/Home

➜  ~ /usr/libexec/java_home -V
Matching Java Virtual Machines (3):
    17.0.1 (x86_64) "Homebrew" - "OpenJDK 17.0.1" /usr/local/Cellar/openjdk/17.0.1/libexec/openjdk.jdk/Contents/Home
    11.0.12 (x86_64) "Homebrew" - "OpenJDK 11.0.12" /usr/local/Cellar/openjdk@11/11.0.12/libexec/openjdk.jdk/Contents/Home
    1.8.0_312 (x86_64) "Homebrew" - "OpenJDK 8" /usr/local/Cellar/openjdk@8/1.8.0+312/libexec/openjdk.jdk/Contents/Home
/usr/local/Cellar/openjdk/17.0.1/libexec/openjdk.jdk/Contents/Home
```

### 动态设置 Java Home

在动态查找 `Java Home` 基础上，结合自定义环境变量、`alias`，实现动态设置 `$JAVA_HOME`(动态切换 `JDK` 版本)。

找到 `.bash_profile/.bashrc/.zshrc` 等配置文件中的任意一个，添加下面的代码：

```bash
## JAVA
export JAVA_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_8_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_11_HOME=$(/usr/libexec/java_home -v11)
export JAVA_17_HOME=$(/usr/libexec/java_home -v17)

## alias for JAVA
alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'
alias java17='export JAVA_HOME=$JAVA_17_HOME'
```

然后退出终端重新打开，或者重新加载配置文件：

```bash
# 以 .zshrc 为例
source ~/.zshrc
```

在终端中使用 `alias`，设置 `$JAVA_HOME` 环境变量，并查看版本信息：

```bash
➜  ~ java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (build 1.8.0_312-bre_2021_10_20_23_15-b00)
OpenJDK 64-Bit Server VM (build 25.312-b00, mixed mode)

➜  ~ java11
➜  ~ java -version
openjdk version "11.0.12" 2021-07-20
OpenJDK Runtime Environment Homebrew (build 11.0.12+0)
OpenJDK 64-Bit Server VM Homebrew (build 11.0.12+0, mixed mode)

➜  ~ java17
➜  ~ java -version
openjdk version "17.0.1" 2021-10-19
OpenJDK Runtime Environment Homebrew (build 17.0.1+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.1+0, mixed mode, sharing)
```

### 不足之处，如何改进

通过 `alias` 设置 `$JAVA_HOME` 环境变量，能达到切换 `JDK` 版本，但是没有持久化，仅仅只是会话级别的切换：

- 旧窗口设置时，没有保存版本
- 新窗口打开时，没有读取版本

要做到持久化的话，需要在设置默认版本时，将版本信息保存在本地文件中，新窗口打开时，读取文件中的版本信息来做初始化。

## 其他 JDK

### Oracle JDK

最近发布的 `Oracle JDK 17` 可以免费使用，[Java SE Development Kit 17.0.1 downloads][2]

> Since September 2021, Oracle provides the Oracle JDK for Java 17 and later under a free use license for All Users.
>
> 自 2021 年 9 月起，Oracle 在所有用户的[免费使用许可][8]下为 Java 17 及更高版本提供 Oracle JDK。—— [Oracle Java SE Support Roadmap][3]

> The NFTC is the license for Oracle JDK 17 and later releases. Subject to the conditions of the license, it permits free use for all users – even commercial and production use.
>
> NFTC 是 Oracle JDK 17 及更高版本的许可证。 根据许可证的条件，它允许所有用户免费使用——甚至商业和生产用途。—— [What is the new “Oracle No-Fee Terms and Conditions” License (NFTC)?][7]

### 其他 OpenJDK

- [Adoptium OpenJDK][9]
- [MicroSoft OpenJDK][10]
- [Alibaba OpenJDK][11]
- ...

## 注意点

1. `JDK@8` 不支持 `java --version`，只支持使用一个分隔符的 `java -version`
2. `Mac OS X 10.5` 以上才支持 `/usr/libexec/java_home` 命令行工具

## 总结

从大学时候的 `JDK 1.5/1.6` 到现在的 `JDK 17`，我已经记不起是第多少次在网上找怎么安装 `JDK`，在哪里下载最新版 `JDK`。

在写这篇文章之前也犹豫要不要写，因为有很多类似的文章（怎么安装 `JDK`，什么是 `OpenJDK`，什么是 `Oracle JDK` 之类的信息)。

思考之后，还是觉得要自己提炼一遍，追根溯源，加深印象的同时，也是记录下来方便以后自己查阅。

## 参考链接

- [新 Mac 如何优雅地配置 Java 开发环境][14]
- [2021 年 Java 开发者生产力报告][5]
- [Oracle Releases Java 17][1]
- [Oracle Java SE Support Roadmap][3]
- [Important Java Directories on Mac OS X][4]
- [Oracle Java SE Licensing FAQ][7]
- [OpenJDK FAQ][12]

[0]: https://brew.sh/
[1]: https://www.oracle.com/news/announcement/oracle-releases-java-17-2021-09-14/
[2]: https://www.oracle.com/java/technologies/downloads/
[3]: https://www.oracle.com/java/technologies/java-se-support-roadmap.html
[4]: https://developer.apple.com/library/archive/qa/qa1170/_index.html
[5]: https://www.oschina.net/news/130872/java-development-2021
[6]: https://github.com/jenv/jenv
[7]: https://www.oracle.com/java/technologies/javase/jdk-faqs.html
[8]: https://www.oracle.com/downloads/licenses/no-fee-license.html
[9]: https://adoptium.net/
[10]: https://docs.microsoft.com/zh-cn/java/openjdk/download
[11]: https://dragonwell-jdk.io/
[12]: https://openjdk.java.net/faq/
[13]: https://github.com/openjdk/jdk
[14]: https://zhuanlan.zhihu.com/p/298535991
