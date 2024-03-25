---
title: "VSCode 安装 Go 相关工具失败"
date: 2023-02-07T17:37:07+08:00
draft: false
tags: [vscode, go]
---

```
go version go1.19.5 windows/amd64
```

使用 VS Code 安装 Go 插件后，安装关联工具时，输出内容报如下内容，或错误信息中包含类似 `unresponse`、`timeout` 等网络问题相关字样时，

```go
Installing golang.org/x/tools/gopls FAILED
{
 "killed": false,
 "code": 1,
 "signal": null,
 "cmd": "/usr/local/go/bin/go get -v golang.org/x/tools/gopls",
 "stdout": "",
 "stderr": "go: downloading golang.org/x/tools/gopls v0.6.9\ngo: golang.org/x/tools/gopls upgrade => v0.6.9\ngo: downloading golang.org/x/tools v0.1.1-0.20210319172145-bda8f5cee399
...

1 tools failed to install.

gopls: failed to install gopls(golang.org/x/tools/gopls): Error: Command failed: /usr/local/go/bin/go get -v golang.org/x/tools/gopls
go: downloading golang.org/x/tools/gopls v0.6.9
go: golang.org/x/tools/gopls upgrade => v0.6.9
go: downloading golang.org/x/tools v0.1.1-0.20210319172145-bda8f5cee399
go: downloading golang.org/x/sys v0.0.0-20210124154548-22da62e12c0c
go: downloading honnef.co/go/tools v0.1.1
go: downloading golang.org/x/mod v0.4.1
golang.org/x/mod/semver
...

go get golang.org/x/tools/gopls: copying /var/folders/gq/bwl3jmx562x5twchgxvb6mlh0000gn/T/go-build703164122/b001/exe/a.out: open /usr/local/go/bin/gopls: permission denied
 no output
```

可考虑开启 Go Module 模式，并设置代理

```
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.io,direct
```

设置完成后重启 VS Code，按照提示再 `Install All` 即可。

GO111MODULE 是一个用于改变 Go 包引入方式的环境变量，在不同 Go 版本下有不同语义，这里改变它的值不一定起关键作用，如果是类似上述网络相关报错，那么重点应该在于代理。

# Ref

> https://l2m2.top/2020/05/26/2020-05-26-fix-golang-tools-failed-on-vscode/
>
> https://stackoverflow.com/questions/66668506/how-to-solve-vs-code-gopls-command-is-not-available
