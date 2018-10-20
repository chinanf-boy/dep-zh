---
id: env-vars
title: Environment Variables
---
dep的行为可以通过一些环境变量修改:

-   [`DEPCACHEAGE`](#depcacheage)
-   [`DEPCACHEDIR`](#depcachedir)
-   [`DEPPROJECTROOT`](#depprojectroot)
-   [`DEPNOLOCK`](#depnolock)

环境变量传递给子命令,因此可用于影响vcs(例如`git`)行为.

* * *

### `DEPCACHEAGE`

如果设置为a[持续时间](https://golang.org/pkg/time/#ParseDuration)(例如.`24h`),它将启用从源存储库缓存元数据:

-   已发布版本的列表
-   项目的内容`Gopkg.toml`文件,在特定版本
-   特定版本的项目包和导入树

必须设置持续时间以启用缓存.(在dep的未来版本中,它将默认启用).持续时间用作TTL,但仅用于可变信息,如版本列表.与不可变VCS修订版相关的信息(包和进口;`Gopkg.toml`声明无限期地缓存.

缓存存在于`$DEPCACHEDIR/bolt-v1.db`,其中版本号是与特定数据模式dep使用相关联的内部编号.

该文件可以安全删除;数据库将根据需要自动重建.

### `DEPCACHEDIR`

允许用户为dep指定自定义目录[本地缓存](glossary.md#local-cache)原始VCS源代码库.默认为`$GOPATH/pkg/dep`.

### `DEPPROJECTROOT`

如果设置,则此变量的值将被视为[项目根](glossary.md#project-root)的[当前的项目](glossary.md#current-project),取代基于GOPATH的推理.

如果您不使用该标准,这主要是有用的`go`工具链作为编译器(例如,使用Bazel),因为在GOPATH之外操作没有多大用处.

### `DEPNOLOCK`

默认情况下,dep创建一个`sm.lock`归档于`$DEPCACHEDIR/sm.lock`为了防止多个dep进程与之交互[本地缓存](glossary.md#local-cache)同时.设置此变量将绕过该保护;不会创建任何文件.这对某些文件系统很有用;特别是VirtualBox共享是行为不端的.
