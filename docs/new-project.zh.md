---
id: new-project
title: 创建一个新项目
---

一旦你有[安装了 dep](installation.md), 我们需要为我们的项目选择一个根目录.这主要是关于导入正确的根路径,以及接下来的`$GOPATH/src`相应目录,定位最后你的项目的落脚点.有四种基本可能性:

1.  现在或最终可能与其他 项目/人员 共享或导入的项目.在这种情况下,选择与其预期网络位置的 VCS 根对应的导入路径,例如,`$GOPATH/src/github.com/golang/dep`.
2.  一个完全本地的项目 - 你无意推送到中央服务器(如 GitHub).在这种情况下,`$GOPATH/src`下面的任何子目录都可以.
3.  需要存在于大型存储库中的项目,例如公司 monorepo. 可能,但会变得更复杂.(不幸的是,还没有关于此的文档 - 即将推出!)
4.  将整个 GOPATH 视为一个单独的项目,这$GOPATH/src`是根.Dep[目前不支持此功能](https://github.com/golang/dep/issues/417)- 它需要一个非空的导入路径，来作为项目导入命名空间的根.

我们将假设第一种情况,因为它是最常见的情况.创建并移入目录:

```bash
$ mkdir -p $GOPATH/src/github.com/me/example
$ cd $GOPATH/src/github.com/me/example
```

现在,我们将初始化项目:

```bash
$ dep init
$ ls
Gopkg.toml Gopkg.lock vendor/
```

在像这样的新项目中,两文件和`vendor`目录实际为空.

这也是设置版本控制的好时机,例如[git](https://git-scm.com/).虽然 dep 不需要对项目进行版本控制,但它可以更容易地检查正常 dep 操作所做的更改.此外,它基本上是现代软件开发的第一最佳实践!

此时,我们的项目已初始化,我们已准备好开始编写代码.你可以打开一个`.go`在编辑器中存档,并开始乱砍.要么,**创建第一个之后`.go`文件**,你可以继续预先填充你的`vendor`，来包含一些您已经知道的项目目录:

```bash
$ dep ensure -add github.com/foo/bar github.com/baz/quux
```

现在你准备继续前进到[每日 Dep](daily-dep.zh.md)!
