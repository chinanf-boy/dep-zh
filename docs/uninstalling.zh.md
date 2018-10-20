---
title: 卸载 Dep
---
## 卸载

要卸载`dep`本身,请遵循这些说明,具体取决于您的安装方式`dep`本来.

### 如果你安装了`dep`通过执行`install.sh`脚本通过卷曲

如果你安装了`dep`使用`install.sh`只需删除已安装的二进制文件即可.

在Linux和MacOS上,`install.sh`脚本安装预编译的二进制文件`$GOPATH/bin/dep`.简单地说是安全的`rm`已安装`$GOPATH/bin/dep`文件:

```sh
$ rm $GOPATH/bin/dep
```

在Windows上,`install.sh`脚本安装预编译的二进制文件`$GOPATH/bin/dep.exe`.只需删除此文件即可安全卸载`dep`.

### 如果你安装了`dep`在MacOS上使用Homebrew

如果你安装了`dep`在MacOS上使用Homebrew,卸载`dep`也使用Homebrew:

```sh
$ brew uninstall dep
```

### 如果你安装了`dep`运用`pacman`在Arch Linux上

如果你安装了`dep`运用`pacman`在Arch Linux上,卸载`dep`像这样:

```sh
$ pacman -R dep
```
