---
title: Git忽略已经过的本地文件并删除线上已经提交的文件
tags:
  - GIT
  - 奇淫技巧
categories:
  - 奇淫技巧
date: '2018-10-26 11:43'
translate_title: git-ignores-local-files-that-have-passed-and-deletes-files-submitted-online
---

在根目录添加`.gitignore`忽略文件，并且设置要忽略的文件或者目录，当要忽略的文件以及提交时

* 删除本地缓存中的文件
```bash
    git rm -r --cached 文件名
```

* 将删除的文件添加到本地暂存区
```bash
    git add .
```

* 将本地变更信息提交到本地仓库
```bash
    git commit -am '变更信息'
```

* 推送到远程仓库
```bash
    git push
```



