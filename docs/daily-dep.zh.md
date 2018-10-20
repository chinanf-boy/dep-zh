---
title: Daily Dep
---
本指南介绍了dep的日常使用.但是,如果你还没有设置Go项目,那么就可以了[创建一个新项目](new-project.md)第一.

Dep是您在正常Go开发过程中经常使用的工具.定期,但简单地说 - 依赖管理永远不是我们想要花费时间或精力的地方!为了与Go的最小化旋钮的理念保持一致,dep具有稀疏的界面;只有两个命令可能会定期运行:

-   `dep ensure`是主要的workhorse命令,并且是唯一一个更改磁盘状态的命令.
-   `dep status`报告您的项目状态,以及Go软件项目的可见范围.

本指南主要以中心为中心`dep ensure`,因为这是您运行以对项目进行更改的命令.该[模型和机制](ensure-mechanics.md)参考文档详细说明了如何在引擎盖下工作,如果您遇到困惑,则值得一读`dep ensure`行为(或只是好奇!).

## 基本

让我们从单词开始吧!

Dep的主要命令是`dep ensure`.动词是"确保"暗示该动作不仅仅是一些单独的,离散的动作(比如添加一个依赖),而是强制执行某种更广泛的保证.如果我们想表达的话`dep ensure`保证作为一个句子,它将是这样的:

> 嘿dep,请确保[我的项目](glossary.md#current-project)是[同步中](glossary.md#sync)那个[`Gopkg.lock`](Gopkg.lock.md)满足我项目中的所有导入以及中的所有规则[ `Gopkg.toml`](Gopkg.toml.md), 然后`vendor/`包含什么`Gopkg.lock`说它应该."

正如叙述所示,`dep ensure`是一个整体运作.而不是提供一系列连续运行的命令,逐步实现某些最终状态,每次运行`dep ensure`根据项目的当前状态提供安全,完整且可重现的依赖关系集.你可能会想象一下`dep ensure`有点像青蛙,从一个百合垫跳到下一个.

## 运用`dep ensure`

你有四次跑步`dep ensure`:

-   添加新依赖项
-   更新现有依赖项
-   要在项目中第一次导入包后删除,要么删除项目中包的最后一次导入
-   赶上改变规则`Gopkg.toml`

如果您不确定进口是否有变化或`Gopkg.toml`规则,运行`dep check`.它会告诉你项目中的不同步.如果有任何不同步,请运行`dep ensure`将它带回来.

让我们探讨这些时刻.要发挥作用,你需要`cd`进入一个已经建立的项目`dep init`.如果您还没有这样做,请查看指南[新项目](new-project.md)和[迁移](migrating.md).

### 添加新的依赖项

假设我们想引入新的依赖关系`github.com/pkg/errors`.这可以通过一个命令完成:

```bash
$ dep ensure -add github.com/pkg/errors
```

> 很像git,`dep status`和`dep ensure`也可以从项目根目录的任何子目录运行(由a的存在决定`Gopkg.toml`文件).

这应该成功,导致更新`Gopkg.lock`和`vendor/`目录,以及为.注入最佳猜测版本约束`github.com/pkg/errors`进入我们的`Gopkg.toml`.但是,它也会报告一个警告:

```bash
"github.com/pkg/errors" is not imported by your project, and has been temporarily added to Gopkg.lock and vendor/.
If you run "dep ensure" again before actually importing it, it will disappear from Gopkg.lock and vendor/.
```

正如警告所示,你应该介绍一个`import "github.com/pkg/errors"`在你的代码中,越快越好.如果你不这样做,以后再做`dep ensure`run会将新添加的依赖项解释为未使用,并自动将其删除`Gopkg.lock`和`vendor/`.这也意味着如果要一次添加多个依赖项,则需要在单个命令中执行此操作,而不是一个接一个地执行:

```bash
$ dep ensure -add github.com/pkg/errors github.com/foo/bar
```

Dep以这种方式工作,因为它将通过对项目代码的静态分析发现的import语句视为必须存在哪些依赖项的规范指示符.这个选择确实增加了一些痛苦,但它减少了摩擦并在其他地方自动清理.权衡!

当然,鉴于这种模式,你没有*不得不*使用`dep ensure -add`添加新的依赖项 - 您也可以添加适当的依赖项`import`代码中的语句,然后运行`dep ensure`.但是,这种方法并不总能很好地发挥作用[`goimports`](https://godoc.org/golang.org/x/tools/cmd/goimports),也不会附加一个`[[constraint]]`成`Gopkg.toml`.尽管如此,它有时也很有用,通常用于快速迭代和袖口试验.

该[确保机械部分`-add`](ensure-mechanics.md#add)有一个更彻底的探索,包括一些方法`dep ensure -add`的行为会根据项目的状态而略有不同.

### 更新依赖项

理想情况下,将依赖项目更新为较新版本是一个命令:

```bash
$ dep ensure -update github.com/foo/bar
```

这也可以在没有参数的情况下尝试更新所有依赖项(尽管通常不推荐):

```bash
$ dep ensure -update
```

`dep ensure -update`搜索与.一起使用的版本`branch`,`version`, 要么`revision`约束中的约束`Gopkg.toml`.这些约束类型具有不同的语义,其中一些允许`dep ensure -update`有效地找到一个"更新"的版本,而其他人将需要手动更新`Gopkg.toml`.该[确保机制](ensure-mechanics.md#update-and-constraint-types)指南更详细地解释了这一点,但如果你想知道什么效果`dep ensure -update`可能有一个特定的项目,`LATEST`在`dep status`输出会告诉你.

### 添加和删​​除`import`声明

如中所述[有关添加依赖项的部分](#adding-a-new-dependency),dep依赖于`import`代码中的语句,用于确定项目实际需要的依赖项.因此,当您添加或删除import语句时,dep通常需要关心它.

只有在发生以下情况之一时才会出现问题`dep ensure`必须使项目恢复同步:

1.  你已经添加了第一个`import`一个包,但已经`import`该项目的其他包.
2.  你已经删除了最后一个`import`一个包,但仍然`import`该项目的其他包.
3.  你已经添加了第一个`import`特定项目中的任何包.(注意:这是[替代添加方法](#adding-a-new-dependency))
4.  你已经删除了最后一个`import`来自特定项目的包.

简而言之,dep涉及整个项目中的一组唯一导入路径,并且仅在进行添加或删除该集合的导入路径的更改时才会关注.`dep check`将快速报告任何此类问题,这些问题将通过运行解决`dep ensure`.

### 规则变化`Gopkg.toml`

`Gopkg.toml`文件包含五种基本类型的规则.该[`Gopkg.toml`文档](Gopkg.toml.md)详细解释它们,但这里是一个概述:

-   `required`,大多相当于`import`的陈述`.go`文件,除了可以列出一个`main`包这里
-   `ignored`,导致dep到黑洞的导入路径(以及它唯一引入的任何导入)
-   `[[constraint]]`,表示版本约束的节和基于每个项目依赖性的一些其他规则
-   `[[override]]`,stanzas相同`[[constraint]]`除了只有当前项目可以表达它们并取代它们`[[constraint]]`在当前项目和依赖项中
-   `[prune]`,全局和每个项目规则,用于管理应从中删除的文件类型`vendor/`

对这些规则中的任何一个的更改可能需要进行更改`Gopkg.lock`和`vendor/`;一次成功`dep ensure`run会立即合并所有这些更改,使您的项目重新同步.

## 可视化依赖关系

通过管道输出来生成依赖树的可视化表示`dep status -dot`至[graphviz的](http://www.graphviz.org/).

### Linux的

```
$ sudo apt-get install graphviz
$ dep status -dot | dot -T png | display
```

### 苹果系统

```
$ brew install graphviz
$ dep status -dot | dot -T png | open -f -a /Applications/Preview.app
```

### 视窗

```
> choco install graphviz.portable
> dep status -dot | dot -T png -o status.png; start status.png
```

![status graph](assets/StatusGraph.png)

## 关键要点

以下是本指南的主要内容:

-   `dep check`将快速报告您的项目的任何方式[同步](glossary.md#sync).
-   `dep ensure -update`是更新依赖项的首选方法,但对于不发布semver版本的项目效率较低.
-   `dep ensure -add`通常是引入新依赖项的最简单方法,但您也可以添加新的依赖项`import`语句然后运行`dep ensure`.
-   如果您曾进行过手动更改`Gopkg.toml`,最好跑`dep ensure`确保一切都同步.
-   `dep ensure`跑步几乎从来都不是错误的;如果您不确定发生了什么,运行它将使您恢复安全("最近的百合花"),或者无法提供信息.

此外,还有其他一些杂项花絮:

-   与Go工具链一样,请避免在自己的项目中使用符号链接.dep容忍了一点,但像Go工具链本身一样,通常不会非常支持符号链接.
-   切勿直接编辑任何内容`vendor/`;dep将无条件地覆盖此类更改.如果您需要修改依赖项,请将其分叉并正确执行.
