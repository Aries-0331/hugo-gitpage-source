---
title: "在新环境中配置多 Git 仓库的 ssh 权限"
date: 2022-11-11T16:21:12+08:00
draft: false
categories: [Git]

---

# 配置多个 Git 仓库的 ssh 权限



1. 跳转至 ssh 配置路径，通常默认为 `~/.ssh`
2. 创建 ssh 密钥对，按照提示创建文件名（eg:repo1-deploy-key、repo2-deploy-key），简易命令用 `./ssh-keygen` 即可，默认创建 rsa 密钥对，也可以通过参数自定义，例如 `./ssh-keygen -t ed25519  -C  username` 等
3. eval `ssh-agent -s`
4. ssh-add repo1-deploy-key repo2-deploy-key
5. 复制 filename.pub 中的内容
6. 粘贴至 Git 仓库的 Settings--Deploy Keys--Add deploy key 中
7. 修改`~/.ssh/config`，参考如下

```
Host repo1.github.com
        HostName github.com
        AddKeysToAgent yes
        IdentityFile ~/.ssh/repo1-deploy-key

Host repo2.github.com
        HostName github.com
        AddKeysToAgent yes
        IdentityFile ~/.ssh/blog-repo
```

8. 分别修改 repo1、repo2 目录下 `.git/config` 中的url，参考如下

```c++
//url = git@github.com:username/repo1.git
url = git@repo1.github.com:username/repo1.git
```

## Failed to connect to github.com port 443: Timed out

将 .git/config 文件中的 https url 修改为 Git 仓库的 ssh rul。

<!--more-->
