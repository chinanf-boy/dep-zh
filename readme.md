# golang/dep [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 `dep`是 Go 的依赖管理工具.它需要 Go 1.9 或更新版本才能编译. 」

[中文](./readme.md) | [english](https://github.com/golang/dep)

---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'golang/dep' -->
<!-- commit = '22125cfaa6ddc71e145b1535d4b7ee9744fefff2' -->
<!-- time = '2018-10-03' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2018-10-03 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/golang/dep.svg
[commit]: https://github.com/golang/dep/tree/22125cfaa6ddc71e145b1535d4b7ee9744fefff2

<!-- doc-templite END generated -->

- [x] readme
- [ ] [docs/**]

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

<p align="center"><img src="docs/assets/DigbyShadows.png" width="360"></p>
<p align="center">
  <a href="https://travis-ci.org/golang/dep"><img src="https://travis-ci.org/golang/dep.svg?branch=master" alt="Build Status"></img></a>
  <a href="https://ci.appveyor.com/project/golang/dep"><img src="https://ci.appveyor.com/api/projects/status/github/golang/dep?svg=true&branch=master&passingText=Windows%20-%20OK&failingText=Windows%20-%20failed&pendingText=Windows%20-%20pending" alt="Windows Build Status"></a>
  <a href="https://goreportcard.com/report/github.com/golang/dep"><img src="https://goreportcard.com/badge/github.com/golang/dep" /></a>
</p>

## Dep

`dep`是 Go 的依赖管理工具.它需要 Go 1.9 或更新版本才能编译.

`dep`是"官方实验".Go 工具链,从 1.11 开始,已经(通过实验)采用了一种与`dep`大相径庭的方法.结果是,我们正在继续开发`dep`，但是,考察与开发dep的工作主要作为之后开发工具链中，版本化行为的替代原型.

有关`dep`的指南和参考资料,查看[文件](https://golang.github.io/dep).

## 安装

强烈建议您使用已发布的版本.发布二进制文件可在[发布](https://github.com/golang/dep/releases)页.

在 MacOS 上,您可以使用 Homebrew 安装或升级到最新发布的版本:

```sh
$ brew install dep
$ brew upgrade dep
```

在其他平台上,您可以使用`install.sh`脚本:

```sh
$ curl https://raw.githubusercontent.com/golang/dep/master/install.sh | sh
```

默认情况下，它将安装到您的`$GOPATH/bin`目录，或您使用指定`INSTALL_DIRECTORY`环境变量的任何其他目录.

如果您的平台不受支持,您需要手动构建它或让团队知道,我们会考虑将您的平台添加到发布版本中.

如果你对 hacking `dep`感兴趣,你可以通过安装`go get`:

```sh
go get -u github.com/golang/dep/cmd/dep
```

## 反馈

非常感谢您的反馈.在此阶段,维护者最感兴趣的是，以工具的用户体验(UX)为中心的反馈.您是否拥有该工具支持的工作流程,或者根本不支持? 任何命令都有令人惊讶的效果,输出或结果吗? 通过提交问题告诉我们,描述您做了什么或想做什么,您期望发生什么,以及实际发生了什么.

## 贡献

贡献非常感谢.维护人员积极管理issue列表,并尝试突出适合新手的问题.该项目遵循典型的 GitHub 拉取请求模型.看[CONTRIBUTING.md](CONTRIBUTING.zh.md)中的更多细节.在开始任何工作之前, 请先评论现有问题,或提交新问题.
