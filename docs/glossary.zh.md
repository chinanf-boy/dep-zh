---
id: glossary
title: Glossary
---
DEP使用一些专门术语.在这里学习!

-   [原子](#atom)
-   [缓存锁定](#cache-lock)
-   [约束](#constraint)
-   [当前项目](#current-project)
-   [可推断的](#deducible)
-   [演绎](#deduction)
-   [直接依赖](#direct-dependency)
-   [外部进口](#external-import)
-   [全球定位系统](#gps)
-   [本地缓存](#local-cache)
-   [锁](#lock)
-   [显示](#manifest)
-   [元数据服务](#metadata-service)
-   [重写](#override)
-   [项目](#project)
-   [项目根](#project-root)
-   [求解器](#solver)
-   [来源](#source)
-   [源根](#source-root)
-   [同步](#sync)
-   [传递依赖](#transitive-dependency)
-   [供应商验证](#vendor-verification)

* * *

### 原子

原子是特定版本的源.在实践中,这意味着一个二元组[项目根](#project-root)和版本,例如`github.com/foo/bar@master`. 原子主要是内部的.[求解器](#solver)这个词在别处很少使用.

### 缓存锁定

也就是"缓存锁定文件".`sm.lock`,用于确保只有一个DEP过程运行在[本地缓存](#local-cache)同时,DEP当前对多个进程进行访问本地缓存的设计是不安全的.

### 约束

约束既有狭义又有较松散的含义.狭义指的是[`[[constraint]]`](Gopkg.toml.md#constraint)中节`Gopkg.toml`. 然而,在某些上下文中,这个词可以更宽泛地用来指将一般规则和要求应用于依赖关系管理的思想.

### 当前项目

DEP正在运作的项目-写它`Gopkg.lock`并填充它的`vendor`目录.

也称为"根项目".

### 可推断的

是否导入路径的速记方式[演绎](#deduction)对于给定的导入路径,将成功返回.也经常使用"不可推断",以引用演绎失败的导入路径.

### 演绎

演绎是确定与源根对应的导入路径子集的过程.某些模式是先验已知的(静态的);其他模式必须通过网络请求(动态)来发现.参见参考文献[导入路径推导](deduction.md)具体情况.

### 直接依赖

一个项目的直接依赖是*进口*从其一个或多个包中,或包含在其[`required`](Gopkg.toml.md#required)列入`Gopkg.toml`.

如果每个字母都在`A -> B -> C -> D`表示仅包含单个包的不同项目,以及`->`指示导入语句,然后`B`是`A`是直接依赖,而`C`和`D`是[传递依赖](#transitive-dependency)属于`A`.

DEP只包含了`required`规则从[当前项目](#current-project) `Gopkg.toml`. 因此,如果`=>`代表`required`而不是一个标准的进口,`A -> B => C`然后`C`是直接依赖的`B` *只有*什么时候?`B`是当前项目.因为`B`- -`C`链接不存在时`A`是当前项目`C`实际上不会出现在图表中.

### 外部进口

安`import`语句,指向项目中的包,而不是源于它的包.例如,一个`import`包内`github.com/foo/bar`如果它指向任何东西,将被认为是外部输入.*其他*比STDLIB或`github.com/foo/bar/*`.

### 全球定位系统

缩写为"去包装解决方案",它是[DEP中的库风格包的子树](https://godoc.org/github.com/golang/dep/gps)是DEP建立的发动机.最常被称为"GPS".

### 本地缓存

DEP维护自己的、原始的上游源集合(一般来说,GIT存储库克隆).这是分开的.`$GOPATH/src`这样就没有维护磁盘状态的义务.`$GOPATH`因为DEP经常需要改变磁盘状态才能完成它的工作.

默认情况下,本地缓存位于`$GOPATH/pkg/dep`. 如果你有多重`$GOPATH`条目,DEP将使用进程的工作目录的逻辑父对象.或者,可以通过[`DEPCACHEDIR`环境变量](env-vars.md#depcachedir).

### 锁

一个通用术语,用于许多语言包管理器,用于DEP保持的信息类型.`Gopkg.lock`文件.

### 显示

一个通用术语,用于许多语言包管理器,用于DEP保持的信息类型.`Gopkg.toml`文件.

### 元数据服务

HTTP服务,当它接收到包含HTTP请求的HTTP请求时,`go-get=1`在查询字符串中,将请求的路径部分解释为导入路径,并通过在HTML中嵌入数据来进行响应.`<meta>`指示底层源根的类型和URL的标记.这是动态的服务器端组件.[演绎](#deduction).

元数据服务的行为定义在[远程导入路径上的文档](https://golang.org/cmd/go/#hdr-Remote_import_paths).

各种引用为"HTTP元数据服务","`go-get`HTTP元数据服务","`go-get`服务等.

### 重写

重写是[`[[override]]`](Gopkg.toml.md#override)中节`Gopkg.toml`.

### 项目

一个项目是一个GO包的树.不能嵌套项目.见[项目根](#project-root)有关如何确定树的根的更多信息.

### 项目根

项目的根导入路径.项目根定义为:

-   对于当前项目,位置`Gopkg.toml`文件定义项目根目录
-   对于依赖关系,网络的根[来源](#source)(VCS存储库)被视为项目根.

这些通常是同一个,但并不总是如此.当使用DEP内部的MMOREPO,多重`Gopkg.toml`文件可能存在于离散项目的子路径中,将每个导入路径指定为项目根目录.当直接在这些项目上工作时,这很好.但是,如果存储库中没有哪个项目试图导入monorepo,dep会将monorepo视为一个大型项目,根目录是Project Root;它将忽略任何和所有`Gopkg.toml`子目录中的文件.

这也可以称为"导入根"或"根导入路径".

### 求解器

"求解器"是对特定于域的SAT求解器的引用.[gps](#gps). 更多细节可以在其[参考页](the-solver.md).

### 来源

保存版本代码的远程实体.源是包含代码的实体,而不是代码本身的任何特定版本.

"源"被用来代替"VCS",因为GO包管理工具将很快学会使用不仅仅是VCS系统.

### 源根

导入路径的一部分,对应于源的网络位置.这类似于[项目根](#project-root),但严格地引用第二个面向网络的定义.

### 同步

DEP是围绕四个状态之间定义良好的关系来设计的:

1.  `import`语句`.go`文件夹
2.  `Gopkg.toml`
3.  `Gopkg.lock`
4.  这个`vendor`目录

如果该关系的任何方面未实现(例如,有一个`import`没有反映在`Gopkg.lock`,或者一个缺少的项目`vendor`然后DEP认为该项目"不同步".

这一概念在[确保力学](ensure-mechanics.md#staying-in-sync).

### 传递依赖

项目的传递依赖项是指它不导入自身,而是通过其依赖项之一导入的依赖项.

如果每个字母都在`A -> B -> C -> D`表示仅包含单个包的不同项目,以及`->`指示导入语句,然后`C`和`D`是`A`的传递依赖关系,而`B`是一个[直接依赖](#transitive-dependency)属于`A`.

### 供应商验证

DEP保证`vendor/`通过对每个项目的内容进行哈希并存储结果,准确地包含预期的代码.[Gopkg.lock文摘](Gopkg.lock.md#digest). 这个摘要是计算出来的.*之后*应用修剪规则.

摘要用于确定内容是否为`vendor/`需要在A期间再生`dep ensure`跑,和`dep check`使用它来确定是否`Gopkg.lock`和`vendor/`在[同步](#sync). 这个[`noverify`](Gopkg.toml.md#noverify)列入`Gopkg.toml`可以用来绕过大多数这些验证行为.
