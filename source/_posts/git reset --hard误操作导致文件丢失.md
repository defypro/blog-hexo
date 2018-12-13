---
title: git reset --hard误操作导致文件丢失
tags:
  - GIT
url: 136.html
id: 136
comments: false
categories:
  - 奇淫技巧
translate_title: git-reset----file-loss-due-to-hard-misoperation
date: 2018-09-18 10:18:30
---

解决已经add，但是没有commit，直接运行git reset --hard xxxxx的情况(因为据说没有add直接返回版本就是死了，而且都硬了的那种)

1.  执行 git fsck --lost-found；
2.  在.git/lost-found目录下找找看有没有你丢失的文件；
3.  有的话复制出来，如果是文本，直接改成正确扩展名，你就笑了；
4.  没有的话，就再去Google吧；

### 相关文章

[关于git reset --hard导致文件丢失的血的教训](https://blog.csdn.net/lijiafa/article/details/78275936 "关于git reset --hard导致文件丢失的血的教训")
