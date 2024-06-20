---
title: "Git 修改 commit 信息"
date: 2021-07-20T15:19:14+08:00
draft: false
categories: [Devtool]
---

- 修改用户名/邮箱

```
1. git rebase -i "commit id" //commit id 选择目标 commit 上一次的 commit ID

2. 修改 pick 为 edit 后，保存退出

3. git commit --amend --author="username <useremail>"//eg: git commit --amend --author="hahaha <hahaha@gmail.com>"

4. git rebase --continue

ps:如果想修改第一次commit的信息，则步骤1中参数 commit id 改为 --root，即 git rebase -i --root
```
