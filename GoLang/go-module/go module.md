# Go Modules

[TOC]



## 1、问题

1. 环境变量goproxy中的direct是有什么作用？
2. go.sum文件有什么作用？
3. go mod中的indirect是什么意思？
4. 为什么一个未使用的模块会被go mod引用了？
5. 如何查看模块的依赖关系？是否有图形可视化工具？
6. 当一个仓库没有发布release和tag时，怎么设置go mod的version？
7. 如何指定一个仓库的commit为版本号？



## 2、Go Modules的历史

### 2.1 GOPATH管理

在使用 GOPATH 模式下，我们需要将应用代码存放在固定的`$GOPATH/src`目录下，并且如果执行`go get`来拉取外部依赖会自动下载并安装到`$GOPATH`目录下。



### 2.2 Dep包管理 

Dep是为Go实现包管理器的官方尝试方案。 从Go1.9版本支持。截至2020年，Dep已被弃官方放弃，取而代之的是Go mod模块。Dep有关更多详细信息，请参阅https://golang.github.io/dep/docs/introduction.html。github地址：https://github.com/golang/dep。



### 2.3 Versioned Go (vgo)

vgo发布于2018年2月，vgo目前只能运行在go1.10之上。

官方觉得dep 的一些细节似乎不适合 Go，因此官方采取了另外的方案来推进Go的版本管理，即 vgo（Go modules的前身），最终演变为我们现在所见到的 Go modules，也在 Go1.11 正式进入了 Go 的工具链。



## 3、Go Modules简介

Go modules 是 Go 语言的依赖解决方案，发布于 Go1.11，成长于 Go1.12，丰富于 Go1.13，正式于 Go1.14 推荐在生产上使用。

Go1.11 是在 2018 年 08 月 25 日，Go Moudle 1.11版本开始支持，为了替代GOPATH。请参阅https://golang.org/ref/mod

| 说明         | 配置                                              |
| ------------ | ------------------------------------------------- |
| 开关环境变量 | GO111MODULE                                       |
| 辅助环境变量 | GOPROXY、GONOPROXY、GOSUMDB、GONOSUMDB、GOPRIVATE |
| 主要文件     | go.mod 和 go.sum                                  |
| 命令行       | go mod xxx                                        |



### 3.1 Go Modules的环境变量

#### 3.1.1 GO111MODULE

GO111MODULE 这个命名代表着Go语言在 1.11 版本添加的，针对 Module 的变量。

Go语言提供了 GO111MODULE 这个环境变量来作为 Go modules 的开关，其允许设置以下参数：

- auto：只要项目包含了 go.mod 文件的话启用 Go modules，目前在 Go1.11 至 Go1.14 中仍然是默认值。
- on：启用 Go modules，1.15版本版本开始为默认值。
- off：禁用 Go modules。



#### 3.1.2 GOPROXY/GONOPROXY

主要是用于设置 Go 模块代理（Go module proxy）。其作用是用于能够脱离传统的GIT方式拉取代码，直接通过镜像站点来快速拉取。



**GOPROXY 配置**

默认值：`https://proxy.golang.org,direct`

GOPROXY的值是一个以英文逗号 “,” 分割的 Go 模块代理列表，允许设置多个模块代理。



> direct

"direct" 为特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取(比如 GitHub)代码。

当值列表中上一个 Go module proxy 返回 404 或 410 错误时，Go 自动尝试列表中的下一个，遇见 “direct” 时回源，遇见 EOF 时终止并抛出类似 “invalid version: unknown revision...” 的错误。



**国内代理**

| 环境配置                                                     | 代理说明 |
| ------------------------------------------------------------ | -------- |
| GOPROXY=https://goproxy.io                                   | 全球代理 |
| GOPROXY=https://goproxy.cn                                   | 七牛云   |
| GOPROXY=https://mirrors.aliyun.com/goproxy                   | 阿里云   |
| GOPROXY=https://goproxy.baidu.com<br/>GOPROXY=https://goproxy.bj.bcebos.com | 百度     |

**goproxy.cn**

github地址：https://github.com/goproxy/goproxy

文档：https://pkg.go.dev/github.com/goproxy/goproxy



**GONOPROXY**

不走代理的依赖模块，1.13版本新增

```shell
go env -w GONOPROXY=\*\*.baidu.com\*\*              # 配置GONOPROXY环境变量,所有百度内代码,不走代理
go env -w GONOSUMDB=\*                              # 配置GONOSUMDB,暂不支持sumdb索引
```



#### 3.1.3 GOSUMDB/GONOSUMDB

Go 1.13 新推出了一个 GOSUMDB，默认值是 sum.golang.org。

GOSUMDB 的全称为 Go CheckSum Database，用来下载的包的安全性校验问题。包的安全性在使用 GoProxy 之后更容易出现，比如我们引用了一个不安全的 GoProxy 之后然后下载了一个不安全的包，这个时候就出现了安全性问题。对于这种情况，可以通过 GOSUMDB 来对包的哈希值进行校验。当然如果想要关闭哈希校验，可以将 GOSUMDB 设置为 off；如果要对部分包关闭哈希校验，则可以将包的前缀设置到环境变量中 GONOSUMDB 中。



关于 GOSUMDB 的配置格式为：<db_name>+\<publickey>+\<url>。

```shell
GOSUMDB="sum.golang.org"
GOSUMDB="sum.golang.org+<publickey>"
GOSUMDB="sum.golang.org+<publickey> https://sum.golang.org"
```

上面三种配置都是合理的，因为对于 http://sum.golang.org，Go 自己知道其对应的 publickey 和 url，所以我们只要配置一个名字即可，对于另外一个 http://sum.golang.google.cn 也是一样。除此之外的，都需要指明 publickey，url 默认是 https://\<db_name>。



关于 GOSUMDB 更多的详细信息可以参考：[https://golang.org/cmd/go/#hdr-Module_authentication_failures](https://link.zhihu.com/?target=https%3A//golang.org/cmd/go/%23hdr-Module_authentication_failures)



**GONOSUMDB**

暂不支持sumdb索引的依赖



#### 3.1.4 GOPRIVATE



### 3.2 go.mod文件

#### 3.2.1 介绍

`go.mod`文件可以通过`require，replace，exclude`语句来说明依赖包的管理规则：

| 参数    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| require | 语句用于指定必不可少的依赖包                                 |
| replace | 语句用于指定需要替换依赖包的地址，比如`golang.org/x/text`包替换成`github.com/golang/text` |
| exclude | 语句用于指定可以忽略依赖项模块                               |



**版本号说明**

1. 项目未打tag

   使用提交号commitID作为版本号，格式如下：`v0.0.0-commit日期-commitID`

   比如 `github.com/beorn7/perks v0.0.0-20180321164747-3a771d992973`

2. 已打tag

   使用正常地用 tag 来作为版本号

   当前分支的tag为v1.3.3。则为 `github.com/DATA-DOG/go-sqlmock v1.3.3`

3. 项目用的 go module 版本是 v2+

   通过让依赖路径带版本号后缀来区分同一个项目里不同版本的依赖，类似于 `gopkg.in/xxx.v2` 的效果。

   使用了 v2+ go module 的项目，项目路径会有个版本号的后缀。

   比如 `github.com/googleapis/gax-go/v2 v2.0.5`



#### 3.2.2 如何使用非语义化版本

**简介**

> 这里以github.com为例，当然其他的仓库也是可以的

命令格式：go get github.com/xxx/xxx@commit-id

具体来说, 特殊的伪版本 `v0.0.0-yyyymmddhhmmss-commit` 引用给定的提交标识符, 它通常是一个缩短的 Git 哈希, 并且必须具有与(UTC)时间戳匹配的提交时间. 这种形式是 `v0.0.0` 预发布的有效语义版本字符串. 例如:

```go
require (
    "google.golang.org/appengine" v1.0.0
    "github.com/google/go-github" v0.0.0-20180116225909-922ceac0585d
)
```

**使用**

```shell
#指定commit
go get github.com/Azure/go-autorest@bca49d5
```

> 会自动将go mod的依赖改为：v0.0.0-commit日期-commitID



#### 3.3.3 indirect[间接]是什么意思



### 3.3 go.sum文件

**格式**

```golang
<module> <version>/go.mod <hash>
```

或者

```golang
<module> <version> <hash>
<module> <version>/go.mod <hash>
```

**说明**

- 其中module是依赖的路径，version是依赖的版本号。hash是以`h1:`开头的字符串，表示生成checksum的算法是第一版的hash算法（sha256）。
- 对于只有 `go.mod` 的 checksum，那么可能是因为对应的依赖没有单独下载。比如用 vendor 管理起来的依赖，便只有 `go.mod` 的 checksum。

- 有些项目实际上并没有 `go.mod` 这个文件，所以 Go 文档里提到这个 `/go.mod` 的 checksum，对于没有 `go.mod` 的项目，Go 会尝试生成一个可能的 `go.mod`，并取它的 checksum。

**功能**

1. 提供分布式环境下的包管理依赖内容校验

   Go 并没有一个中央仓库。发布者在 GitHub 上给自己的项目打上 0.1 的 tag 之后，依旧可以删掉这个 tag ，提交不同的内容后再重新打个 0.1 的 tag。

   所以只能在每个项目里存储自己依赖到的所有组件的 checksum，才能保证每个依赖不会被篡改。

2. 作为 transparent log 来加强安全性

   go.sum 还有一个很特别的地方，就是它不仅仅记录了当前依赖的checksum，还保留了历史上每次依赖的 checksum。这种做法效仿了 transparent log 的[概念](https://research.swtch.com/tlog)。

   go.sum 之所以要用 transparent log 的形式记录历史上的每个checksum，是为了便于 sum db 的工作。

**缺点**

容易产生合并冲突

由于许多项目都没有通过打tag的方式来管理发布，每个commit都相当于新发布一个版本，这导致拉取它们的代码时会偶尔往 go.sum 文件里插入一条新记录。

多人合作开发时，同时依赖同一个库的不同commit，插入2条不同的记录。在合并代码的时候 go.sum可能会冲突。



### 3.4 Go mod 命令行

| 命令            | 作用                                                |
| :-------------- | :-------------------------------------------------- |
| go mod init     | 生成 go.mod 文件                                    |
| go mod download | 下载 go.mod 文件中指明的所有依赖                    |
| go mod tidy     | 整理现有的依赖                                      |
| go mod graph    | 查看现有的依赖结构                                  |
| go mod edit     | 编辑 go.mod 文件                                    |
| go mod vendor   | 导出项目所有的依赖到vendor目录/下载依赖包到本地缓存 |
| go mod verify   | 校验一个模块是否被篡改过/校验所有依赖包的正确性     |
| go mod why      | 查看为什么需要依赖某模块/解释为什么需要依赖         |



#### 3.4.1 go get添加依赖包

```shell
# 添加依赖包
go get -v github.com/Azure/go-autorest				# 最新版本
go get -v github.com/Azure/go-autorest@v10.15.5		# 指定版本
```



#### 3.4.2 go mod edit

```shell
# 添加单个依赖包
go mod edit -require=golang.org/x/text								# 使用最新版
go mod edit -require github.com/Azure/go-autorest@v10.15.5			# 指定版本
go mod edit -require github.com/Azure/go-autorest@v10.15.5 -go 1.15	# 指定go版本

# 移除单个依赖包
go mod edit -droprequire=golang.org/x/text

# 忽略依赖项
go mod edit -exclude github.com/Azure/go-autorest/autorest/adal@v0.8.3 -go 1.15 # 忽略指定版本
go mod edit -exclude github.com/Azure/go-autorest/autorest/adal@v0.9.17 -go 1.15
```



#### 3.4.4 go mod tidy

删除未使用依赖，增加使用的依赖。如果go mod文件中未指定的依赖，默认会使用最新版本。



#### 3.4.5 go mod graph如何查看依赖



#### 3.4.6 如何使用可视化工具

1. 源码地址

   https://github.com/poloxue/modv

2. 安装绘图软件

   ```shell
   yum install graphviz
   ```

3. 安装工具

   ```shell
   go install github.com/poloxue/modv
   ```

4. 使用

   ```shell
   go mod graph | modv | dot -T svg -o modv.svg
   ```



### 参考

- [如何实现 Go Module 依赖关系的可视化](https://zhuanlan.zhihu.com/p/88097101)
- [Go mod使用指南](https://studygolang.com/articles/31112)
- [Go Modules 终极入门](https://studygolang.com/articles/26836)
- [尝鲜vgo](https://studygolang.com/articles/12429)
- [谈谈go.sum](http://www.zyiz.net/tech/detail-97723.html)
- [golang 环境变量 GOSUMDB](https://blog.csdn.net/wide288/article/details/105050043)
- [go get 指定commit 版本](https://blog.csdn.net/weixin_44676081/article/details/118070746)

