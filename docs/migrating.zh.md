---
id: migrating
title: 迁移到 Dep
---

理想情况下,将现有GO项目迁移到Dep是很简单的:

```bash
$ cd $GOPATH/src/path/to/project/root
$ dep init
```

对许多项目来说,这就会起作用.`dep init`将对你的依赖项使用哪些版本进行有意义的猜测,生成理智`Gopkg.toml`,`Gopkg.lock`和`vendor/`，如果你的测试通过，并构建成功,那么你可能已经完成了.(如果是的话,祝贺你! 那你的下一步应该去[每日Dep](daily-dep.zh.md)看看)

但一些项目的迁移过程仍会出现困难问题.如果您是第一次尝试dep,这可能会特别令人沮丧,因为您试图同时学习，如何使用dep,和用dep*应*对项目如何管理, 好消息是`dep init`通常就是难度最大的那座山峰; 一旦你跨过它,事情就变得容易多了.

本指南的目的是提供足够的信息,让你对`dep init`发生的事情进行推理.这样,你至少可以理解你遇到的是什么样的问题,你可以采取什么步骤来解决这些问题.为此,我们将首先概述一下`dep init`做了啥.

> 注:第一次运行`dep init`可能需要相当长的时间,因为Dep正在把你所有的依赖者的新克隆创造到一个特殊的位置,`$GOPATH/pkg/dep/sources/`. 这对于Dep的正常操作是必要的,并且在很大程度上仅是一次性成本.

## `dep init`力学

迁移现有项目时,首要目标是`dep init`尽可能自动化创建一个`Gopkg.toml`.这必然是一个启发性的目标, 因为Dep对于你之前所做的一切可能没有1:1的对应关系.因此,重要的是，期待`dep init`能尽力而为, 进行着自动化迁移.

`dep init`行为取决于现有代码库中的内容和传递给它的命令参数.然,它总是分两个阶段进行:

1.  *推理阶段:*从各种来源推理规则和提示,说明使用哪些版本的依赖项.
2.  *求解阶段:*在Dep的模型下设计一个可以接受的解决方案,同时尽可能多地结合上面的推理结论.

### 推理阶段

`dep init`的推理阶段其行为各不相同.默认情况下,`dep init`将查看您的代码库中[其他可以理解的GO包管理工具](https://github.com/golang/dep/tree/master/internal/importers)的元数据文件，以及尝试自动将这些文件中的数据迁移到,dep具有意义的概念中.根据工具和dep找到的特定值,可以将来自工具的元数据视为:

-   一个提示{**hint**}: dep在求解阶段会尽力遵守的信息,但是如果它找不到遵循提示的解决方案,就会丢弃这些信息.
-   一个规则{**rule**}: 在求解阶段必须遵守的信息,最终将作为一个`[[constraint]]`出现在`Gopkg.toml`文件. 如果求解阶段不能找到满足规则的解决方案,那么它会失败，且带有一个消息.

有三种情况可以导致Dep不做任何基于工具的推理:

-   您的项目不使用包管理工具
-   Dep还不支持你使用的工具
-   你告诉它不要,通过`dep init -skip-tools`

在基于工具的推理完成之后,Dep通常会进入求解阶段.但是,如果用户传递`-gopath`标记,Dep将首先尝试通过检查当前项目中包含的GOPATH，来填充从工具元数据中提取的推理中的任何漏洞. 仅仅填充从GOPATH收集到的提示,它们永远不会取代工具元数据的推理. 但如果你想把GOPATH完全负责,就需要通过这两个标志:`dep init -skip-tools -gopath`.

一旦Dep编译了它的推理集合,它就开始求解.

### 求解阶段

一旦推理阶段完成,Dep已装配的规则和提示将被传递给[求解器](the-solver.zh.md)，让求解器设计传出完整的depgraph,最终将被记录为`Gopkg.lock`. `dep ensure`也是相同的求解过程. 并且成功完成它, 意味着dep已经找到遵守所有推理规则的依赖版本的组合,以及尽可能多的推理提示. 如果解决成功,那么辛苦的工作就完成了; 剩下的大部分是写出`Gopkg.toml`,`Gopkg.lock`和`vendor/`.

求解器返回一个解决方案,该解决方案本身就是[存储在一个`Gopkg.lock`中的数据](https://godoc.org/github.com/golang/dep#Lock)的[一个画像](https://godoc.org/github.com/golang/dep/gps#Solution), 是整个依赖图的传递完整、可再现的快照. 从一个解决方案写出`Gopkg.lock`，只不过是一个复制和编码{**copy-and-encode**}操作,和写`vendor/`时，将指定的解决方案中列出的每一个项目，放到指定的修订中.这与`dep ensure`的行为完全一样.

`Gopkg.toml`有点不同. 不能保证为你项目的所有(甚至任何)依赖推理出规则,但我们仍然希望填充`Gopkg.toml`带有些许理智的值. 因此,对于未推理规则的任何依赖项,dep检查解决方案，以查看最终选择了什么版本,并基于此创建约束:

-   如果一个分支,像`master`，在解决方案中被选中。`branch: "master"`将出现在`Gopkg.toml`.
-   如果选择了符合语义版本的版本,像`v1.2.0`,然后将指定为最小版本:`version: "v1.2.0"`.
-   如果只选择原始修订版,则`Gopkg.toml`没有任何内容. 而Dep确实约束了`Gopkg.toml`中的`revision: "…"`，因为使用它们被认为是反模式, 所以dep不会自动创建它们,以避免隐含地鼓励使用它们.

## 处理失败

首先,确保你`dep init`带`-v`.这将提供更多的信息.

`dep init`，通常来说与Dep一样,它既有硬模式又有软故障模式.硬故障导致进程挂起或完全中止,且没有任何东西写入磁盘. 软故障可能包括或不包括警告,但最终写入`Gopkg.toml`,`Gopkg.lock`和`vendor/`-只是都不是你想要的而已。在我们深入研究这些内容之前,让我们来设置一些上下文.

虽然dep贡献者已经投入了巨大的努力来创建到dep的自动迁移路径,但是这些路径仅是尽力而为并且不精确. 从其他工具或GOPATH转换为完全保真并不总对的. dep是一个自以为是的工具,具有相应的自以为是的模型,并且该模型有时确实与其他工具有根本的不同.有时这些模型不匹配导致硬故障,有时是软的,总是充满不确定性.

因为这些都是深层次的假设,他们的症状可以是多样的和令人惊讶的.记住这些假设可以帮你节省一些时间.

- 1. Dep不允许嵌套`vendor/`目录;它在项目的根目录下，将所有依赖项平铺到`vendor/`目录的最顶层. 这是Dep模型的基础,不能被禁用.
- 2. Dep整体控制`vendor`, 并且将吹走任何偏离其上游源Dep选择的手动更改或添加.
- 3. Dep要求来自给定项目/仓库的所有包都在同一版本.
- 4. dep通常不关心GOPATH上的内容; 它只处理源自远程网络位置的项目.(提示推理是唯一的例外;一旦解决阶段开始,GOPATH——以及您对代码所做的任何自定义更改——将被忽略.)
-  5. Dep通常倾向于语义版本标记的发布到分支(当没有给出任何附加规则)时.这是从`go get`还有其他一些工具的"默认分支"模型的重大转变.它可以导致Dep做出惊吓的选择，只因无法推理一个规则的依赖.
-  6. Dep假定所有生成的代码都存在,并且已经提交给源.

迄今为止,很少有项目能够找到一种合理的方法来适应这些要求. 如果你不能想出如何使你的项目适合,请提交一个问题-虽然dep必然不能适应每一个现有的方法,它的目标是定义规则,所有的Go项目可以合理地适应.

### 硬故障

所有的硬故障模式在参考文献中被广泛地覆盖.[失效模式](failure-modes.zh.md).

因为求解器及其所有可能的故障都是相同的.`dep init`以及`dep ensure`有一个单独的部分来理解和处理它们:[解决故障处理](failure-modes.zh.md#solving-failures). `dep init`这可能更棘手，然而, 其实许多补救措施需要调整`Gopkg.toml`.

不幸的是,当`dep init`失败时, 它不会写出部分`Gopkg.toml`.这是一个已知的、关键的问题,[我们有一个开放的问题(需要帮助!)](https://github.com/golang/dep/issues/909).

同时,如果您遇到的特定错误确实需要`Gopkg.toml`调整一下, 不幸的是, 你需要手动完成`dep init`操作: 创建一个空[`Gopkg.toml`](Gopkg.toml.md), 并用手工填充规则. 在重排之前,确保你已经`dep init`具有推理参数的各种组合(`-skip-tools`和`-gopath`), 看看他们能不能给你点东西.

### 软故障

软故障就是`dep init`似乎干净地退出的这种情况,但随后`go build`或`go test`失败.Dep的软故障通常比细微错误大得多,例如,当您尝试构建时,类型错误发生爆炸,因为选择了一些依赖的错误版本.

如果你遇到这样的问题,`dep status`是您的第一个诊断步骤;它将报告为您的所有依赖项选择了哪些版本.很显然,依赖关系仅仅是从build或test错误消息中产生的问题.如果没有,比较`dep status`列出与前一个工具记录的版本,找出差异.

一旦确定了有问题的依赖关系,下一步就是对它们, 在`Gopkg.toml`进行适当的控制..

对于下面的每一项, 都假设在提出建议的改变之后,您应该运行`dep ensure`.如果失败,请教[解决故障处理](<>).

-   如果`[[constraint]]`错了被推理为你的直接依赖之一,改变它.然后,向Dep提交一个问题(请!)-而`dep init`可以选择省略约束,错误地转换可以认为是一个bug.
-   如果您的传递依赖项位于错误的版本中,则把`[[override]]`定义逼到你需要的版本.
    -   如果您需要的版本是特定的Git提交,那么最好在那个项目的`Gopkg.lock`, 手动更改`revision`期望的散列,然后放弃`version`或`branch`字段(如果有的话).
-   如果你的一个直接依赖是错误的版本， 并且`Gopkg.toml`已没有`[[constraint]]`关于它, 那,定义一个合适的.
    -   与传递依赖关系一样,如果需要的版本是特定的Git提交,则最好对`Gopkg.lock`手动操作.

希望这些信息足以帮助您通过项目迁移到dep.如果没有,请随时提交问题,或者加入我们[#vendor on the Gopher's slack](https://gophers.slack.com/messages/C0M5YP9LN)寻求帮助!
