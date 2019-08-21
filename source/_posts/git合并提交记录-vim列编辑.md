---
title: git合并提交记录&vim列编辑
date: 2019-07-17 13:57:38
categories: 
  - git 
  - vim
tags: 
  - git
---



查看日志提交记录

```
$ git log
commit 93c290851b3baebd612885ea5b866311cdd913d0
Author: solooo <pjian108@gmail.com>
Date:   Tue Mar 26 10:25:46 2019 +0800

    add themes

commit 6414ff9530f6006b65e2c765f675a88012402a82
Author: solooo <pjian108@gmail.com>
Date:   Tue Mar 26 10:24:47 2019 +0800

    update

commit 6cf175fe84f28967416f896c7865945930857e78
Author: solooo <pjian108@gmail.com>
Date:   Tue Mar 26 10:16:27 2019 +0800

    update

commit 22960dcfe9e2eab38aef77f002590ef2e39110fb
Author: solooo <pjian108@gmail.com>
Date:   Mon Jan 28 15:41:47 2019 +0800

    create

```
<!-- more -->
找到要合并的记录**之前**的一条commit id

```
22960dcfe
```

使用rebase命令合并

```
$ git rebase -i 22960dcfe
pick 6cf175f update
pick 6414ff9 update
pick 93c2908 add themes
pick 686983a update
pick 37553e5 upgrade dependency

# Rebase 22960dc..37553e5 onto 22960dc (5 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out

```

vim 修改第一列的 **pick** 改为 fix 或 squash 具体区别下方有注释说明

vim 列编辑

```
ctrl+v 进入列模式
方向键选择编辑的列内容
shift+i 进入列编辑模式输入内容
两次 Esc 保存编辑
```

shift+i 进入列编辑模式时，光标只在第一行有，双南 Esc 保存后整列会赋值

以上操作完成后，保存文件即可合并提交记录

切换不同分支合并代码时，看到的记录也只有一条
