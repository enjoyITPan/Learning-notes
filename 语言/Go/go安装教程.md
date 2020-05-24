## mac安装go

1、从https://golang.org/doc/install下载golang安装包，该安装包会将go安装在/usr/local/go目录下，并且将/usr/local/go/bin目录添加到path环境中。

补充：类unix系统环境变量说明

## GOPATH 和 GOROOT

不同于其他语言，**go中没有项目的说法，只有包**, 其中有两个重要的路径，**`GOROOT`** 和 **`GOPATH`**

Go开发相关的环境变量如下：

- GOROOT：GOROOT就是Go的安装目录，（类似于java的JDK）
- GOPATH：GOPATH是我们的工作空间,保存go项目代码和第三方依赖包

**`GOPATH`**可以设置多个，其中，第一个将会是默认的包目录，使用 go get 下载的包都会在第一个path中的src目录下，使用 go install时，在哪个**`GOPATH`**中找到了这个包，就会在哪个`GOPATH`下的bin目录生成可执行文件

***可以将$GOPATH/bin添加到环境变量中，方便执行一些go工具：export PATH="$GOPATH:$PATH"***

GOPATH是开发时的工作目录。用于：

1. 保存编译后的二进制文件。
2. `go get`和`go install`命令会下载go代码到GOPATH。
3. import包时的搜索路径

使用GOPATH时，GO会在以下目录中搜索包：

1. `GOROOT/src`：该目录保存了Go标准库代码。

2. `GOPATH/src`：该目录保存了应用自身的代码和第三方依赖的代码。
## go env

1、GOPROXY=http://goproxy.cn,direct 设置代理

2、GOPRIVATE=*.mingbai.com 私有域名的配置会让go mod 不走代理，而是用版本控制工具（git）的方式去拉去依赖。解决拉内网或者本地代码的问题。

3、

   



