---
title: "Tips"
date: 2022-11-11T16:21:12+08:00
draft: true
categories: [tips]

---

> 在新环境中配置 Git 仓库的 ssh 权限

```
1.cd ~/.ssh
2.ssh-keygen -t ed25519  -C  username //创建 ssh 密钥对，按照提示创建文件名(eg:filename)
3.eval `ssh-agent -s`
4.ssh-add filename
5.复制 filename.pub 中的内容
6.粘贴至 Git 仓库的 Settings--Deploy Keys--Add deploy key 中
```



> Failed to connect to github.com port 443: Timed out

将 .git/config 文件中的 https url 修改为 Git 仓库的 ssh rul。
