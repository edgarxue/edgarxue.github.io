---
title: "git clone 出现error:  while accessing https…… fatal: HTTP request failed的思路及解决"
date: 2018-05-04T16:11:57+08:00
tags: [ "Git","Centos","Linux" ]
categories: [ "Ops" ]
---

>环境：Centos 6.5
>
>git版本： 1.7.1

----

瞎逛github的时候发现[cloudflare](https://github.com/cloudflare)有个alertmanager的皮**unsee**，就准备搞下来试试，竟然遇到如下错误：

```shell
# git clone -vvvvv https://github.com/cloudflare/unsee.git
Initialized empty Git repository in /home/wenba/go/unsee/.git/
error:  while accessing https://github.com/cloudflare/unsee.git/info/refs

fatal: HTTP request failed
```

赶紧搜了一下解决办法，网上一众大神都说要升级git……简直💊啊，如此伤筋动骨，我表示不服……

那就一步一步的来解决吧！

----
### 首先看验证下是不是网络问题吧

```shell
 # wget https://github.com/cloudflare/unsee.git
--2018-05-04 11:17:05--  https://github.com/cloudflare/unsee.git

Resolving github.com... 13.229.188.59, 13.250.177.223, 52.74.223.119

Connecting to github.com|13.229.188.59|:443... connected.

HTTP request sent, awaiting response... 301 Moved Permanently

Location: https://github.com/cloudflare/unsee [following]

--2018-05-04 11:17:06--  https://github.com/cloudflare/unsee

Reusing existing connection to github.com:443.

HTTP request sent, awaiting response... 200 OK
……
```

网络可达，排除墙的问题。

----

### 关闭http.sslVerify试下

```shell
# git config --system http.sslVerify false
```

再试，问题依旧。

#### 检查git clone依赖包是否齐全

```bash
# yum install curl-devel zlib-devel -y
```

再试，问题依旧。

***那么，到这里，就基本可以排除网络和git的问题了！***

### _问题在哪里呢？_

不妨回过头来思考下https，是不是通信组件有问题呢？

nss？openssl？ca-certificates？

那就先检查版本和安装情况吧

```bash
yum list|grep -E "^openssl\.x86|^nss\.x86|ca-certificates"
ca-certificates.noarch                      2013.1.94-65.0.el6           @anaconda-CentOS-201311272149.x86_64/6.5
nss.x86_64                                  3.15.1-15.el6                @anaconda-CentOS-201311272149.x86_64/6.5
openssl.x86_64                              1.0.1e-30.el6_6.5            @updates
ca-certificates.noarch                      2017.2.14-65.0.1.el6_9       updates
nss.x86_64                                  3.28.4-4.el6_9               updates
openssl.x86_64                              1.0.1e-57.el6                base
```

既然如此，升级吧

```bash
yum update ca-certificates nss openssl openssl-devel -y
```

再试一下

```bash
# git clone https://github.com/cloudflare/unsee.git
Initialized empty Git repository in /data/devops/unsee/.git/
remote: Counting objects: 5469, done.
remote: Compressing objects: 100% (58/58), done
……
```

#### __解决!__
















