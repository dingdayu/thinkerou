---
title: 使用 Go vendor
categories: [
  "Go"
]
tags: [
  "Go"
]
date: 2016-04-10
image: "simple.jpg"
---

## Go vendor 介绍

Go 语言在发布 1.5 版本时，就说可以使用自身提供的 `vendor` 特性，但是需要设置如下环境变量：

    GO15VENDOREXPERIMENT=1

在发布 1.6 版本时，该环境变量的值已经默认设置为 1 了，该值可以使用 `go env` 命令查看。

根据官方的说法，在发布 1.7 版本时，将去掉该环境变量，默认开启 `vendor` 特性。

现在也有很多包管理工具，比如 `Godep`、`govendor`、`gvt` 等等，并且也都支持语言本身提供的 `vendor` 特性，那么我的问题是：

> 不使用第三方包管理工具，如何使用 `vendor` 特性呢？

一开始 google 了好多文档都没有符合要求的，最后就在 [stackover flow](http://stackoverflow.com/questions/37237036/how-should-i-use-vendor-in-go-1-6) 上求问才找到解决方法。

#### 1. 设置环境变量

> GOPATH="/Users/thinkerou/xyz/"

#### 2. 建立测试需要的目录

>
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ pwd
    /Users/thinkerou/xyz
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ ls
    src
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ cd src
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ ls
    ou
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ cd ou
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ ls
    main.go vendor
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ cd vendor/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou/vendor$ ls
    vendor.json

就是需要在 `$GOPATH` 目录下建立 `src/my-program-name` 目录，同时需要在目录 `src/my-program-name` 目录下建立 `vendor` 目录。

> 注：vendor.json 暂时还没有用处，应该是只有使用第三方包管理工具时才有用。

#### 3. 编写测试文件

测试的 `main.go` 文件内容为：

>
    package main
    import (
	    "fmt"
	    "net/http"
	    "github.com/zenazn/goji"
	    "github.com/zenazn/goji/web"
    )
    func hello(c web.C, w http.ResponseWriter, r *http.Request) {
	     fmt.Fprintf(w, "Hello, %s!", c.URLParams["name"])
    }
    func main() {
	    goji.Get("/hello/:name", hello)
	    goji.Serve()
    }

示例程序中使用到了第三方 `goji` 库。

#### 4. 获取第三方库文件

使用命令 `go get` 获取依赖的第三方库文件，如下：

>
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ ls
    main.go  vendor.j
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ go get
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ ls
    main.go  vendor.j
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ cd ..
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ ls
    github.com ou
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ cd github.com/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/github.com$ ls
    zenazn
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/github.com$ cd zenazn/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/github.com/zenazn$ ls
    goji
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/github.com/zenazn$ cd goji/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/github.com/zenazn/goji$ ls
    LICENSE            bind               example            graceful           serve_appengine.go
    README.md          default.go         goji.go            serve.go           web

从上可知，在 `$GOPATH/src` 目录下出现了第三方依赖库的目录 `github.com/zenazn/goji/` 树。
 
在获取到第三方库文件后，使用命令 `go build` 就会生成可执行文件，或使用命令 `go run main.go` 直接运行程序。

到这里，会发现还是使用老方法来运行程序的，并没有使用到本文要讨论的话题：**如何使用 vendor 特性？**

#### 5. 使用 vendor 特性

为了使用上新的 `vendor` 特性，需要做一些操作：

>
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ pwd
    /Users/thinkerou/xyz
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ ls
    bin pkg src
    thinkerou@MacBook-Pro-thinkerou:~/xyz$ cd src/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ ls
    github.com ou
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src$ cd ou/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ ls
    main.go vendor
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou$ cd vendor/
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou/vendor$ ls
    vendor.json
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou/vendor$ mv ../../github.com .
    thinkerou@MacBook-Pro-thinkerou:~/xyz/src/ou/vendor$ ls
    github.com  vendor.json

从上可知，使用原来的老方法，在 `$GOPATH` 目录下已经产生了 `bin` 和 `pkg` 目录，都是熟知的。

为了支持 `vendor` 特性，需要将使用 `go get` 命令获取到的第三方目录 `剪切或复制` 到 `src/my-program-name/vendor/` 目录下。

如此，使用命令 `go build` 或 `go run main.go` 即可执行程序，并不会去重新下载依赖的第三方库文件，而是直接使用 `vendor` 目录下的文件。

#### 6. 白话 vendor 原理

简单来说，`vendor` 的原理就是：

> 在执行 `go build` 或 `go run` 命令时，会首先去判断 `vendor` 是否存在，以及是否存在依赖的第三方库文件，如果满足则使用之；否则，就走原来的流程去获取第三方库文件到 `$GOPATH/src/` 目录下。 

## 参考资料

- [How should I use vendor in Go 1.6?](http://stackoverflow.com/questions/37237036/how-should-i-use-vendor-in-go-1-6)

- [PackageManagementTools](https://github.com/golang/go/wiki/PackageManagementTools)

