---
title: CentOS7 DNS错误导致yum无法安装软件
date: 2017-09-19 11:52:48
categories:
- Linux
- technology
tags:
- Linux
---
刚安装好的CentOS 7在安装vim时，提示以下错误：`"Could not resolve host: mirrorlist.centos.org; Unknown error"`

错误详细：
```
[root@ht247 ~]# yum install -y vim
Loaded plugins: fastestmirror
Could not retrieve mirrorlist http://mirrorlist.centos.org/?release=7&arch=x86_64&repo=os&infra=stock error was
14: curl#6 - "Could not resolve host: mirrorlist.centos.org; Unknown error"


 One of the configured repositories failed (Unknown),
 and yum doesn't have enough cached data to continue. At this point the only
 safe thing yum can do is fail. There are a few ways to work "fix" this:

     1. Contact the upstream for the repository and get them to fix the problem.

     2. Reconfigure the baseurl/etc. for the repository, to point to a working
        upstream. This is most often useful if you are using a newer
        distribution release than is supported by the repository (and the
        packages for the previous distribution release still work).

     3. Disable the repository, so yum won't use it by default. Yum will then
        just ignore the repository until you permanently enable it again or use
        --enablerepo for temporary usage:

            yum-config-manager --disable <repoid>

     4. Configure the failing repository to be skipped, if it is unavailable.
        Note that yum will try to contact the repo. when it runs most commands,
        so will have to try and fail each time (and thus. yum will be be much
        slower). If it is a very temporary problem though, this is often a nice
        compromise:

            yum-config-manager --save --setopt=<repoid>.skip_if_unavailable=true

Cannot find a valid baseurl for repo: base/7/x86_64

```
网上找了原因：**DNS配置错误导致无法连接到源**。

解决方法：编辑`/etc/resolv.conf`，如果没有配置则添加以下内容

    nameserver 8.8.8.8

如果DNS已配，则查看防火墙配置，具体不没碰到，以后遇到再补充
