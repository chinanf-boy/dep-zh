---
title: 安装
---

强烈建议您使用dep的已发布版本.虽然tip从未故意破坏,但其稳定性无法保证.

## 二进制安装

预编译的二进制文件可在[发布](https://github.com/golang/dep/releases)页.你可以使用`install.sh`脚本，为您的本地平台自动安装一个:

```sh
$ curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
```

## MacOS

使用Homebrew安装，或升级到最新发布的版本:

```sh
$ brew install dep
$ brew upgrade dep
```

## Arch Linux

安装`dep`包:

```sh
pacman -S dep
```

## 从源码安装

下面的代码片段从源代码安装最新版本的dep, 并将版本设置为二进制文件，让`dep version`可以按预期工作.

请注意,建议不要将此方法用于一般情况. 我们不试图打破tip,但我们也不保证其稳定性.同时,我们热爱愿意接受实验，并向我们提供快速反馈的用户!

```sh
go get -d -u github.com/golang/dep
cd $(go env GOPATH)/src/github.com/golang/dep
DEP_LATEST=$(git describe --abbrev=0 --tags)
git checkout $DEP_LATEST
go install -ldflags="-X main.version=$DEP_LATEST" ./cmd/dep
git checkout master
```

## 开发

如果你想hacking dep,你可以通过安装`go get`:

```sh
go get -u github.com/golang/dep/cmd/dep
```

请注意,dep需要一个正常运行的Go工作空间和GOPATH.如果您对Go工作区和GOPATH不熟悉,请查看[Go语言文档](https://golang.org/doc/code.html#Organization)并设置您的本地工作区. Dep的模型可以在没有GOPATH的情况下工作, 但我们还没有.

## 卸载

正在寻找一种卸载方式`dep`? 有一个单独的[卸载页面](uninstalling.zh.md)说明这个问题!
