---
id: deduction
title: 导入路径的推导
---
演绎是dep的算法,用于查看导入路径并确定路径中与源根对应的部分.该算法具有一个静态组件,通过该组件,一小组已知的、流行的主机(如GitHub和Bitbucket)可以推导出它们的根:

-   `github.com/golang/dep/gps`>`github.com/golang/dep`
-   `bitbucket.org/foo/bar/baz`>`bitbucket.org/foo/bar`

由静态演绎支持的主机集与[那些支持`go get`](https://golang.org/cmd/go/#hdr-Remote_import_paths):

-   吉图布
-   比特桶
-   发射台
-   IBM DeVoPS服务

此外,DEP还处理[GopkGin](http://gopkg.in)直接使用静态演绎是因为,由于内部实现细节,它也是附加过滤器以适应gopkg的版本化语义的最简单方法.这是好的,如Gopkg.在规则中,映射规则本身是静态的.

如果静态逻辑无法识别给定导入路径的根,则算法继续到动态组件:dep对导入路径发出HTTP(S)请求,并且服务器期望发送回嵌入在HTML响应中的根导入路径.同样,这直接模拟了`go get`.

导入路径演绎适用于以下所有内容:

-   `import`所有发现的语句`.go`文件夹
-   导入路径[`required`](Gopkg.toml.md#required)列入`Gopkg.toml`
-   `name`两者的性质[`[[constraint]]`](Gopkg.toml.md#constraint)和[`[[override]]`](Gopkg.toml.md#override)中的诗节`Gopkg.toml`. 这仅仅是为了验证目的,强制执行这些名称仅对应于项目/源根.
