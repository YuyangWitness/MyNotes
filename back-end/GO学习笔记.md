# GO学习笔记

## Go的环境变量

### GO111MODULE

可以用环境变量 `GO111MODULE` 开启或关闭模块支持，它有三个可选值：off、on、auto，默认值是 auto。

- `GO111MODULE=off` 无模块支持，go 会从 GOPATH 和 vendor 文件夹寻找包。
- `GO111MODULE=on` 模块支持，go 会忽略 GOPATH 和 vendor 文件夹，只根据 go.mod 下载依赖。
- `GO111MODULE=auto` 在 $GOPATH/src 外面且根目录有 go.mod 文件时，开启模块支持。

在使用模块的时候，GOPATH 是无意义的，不过它还是会把下载的依赖储存在 GOPATH/pkg/mod 中，也会把go install的结果放在GOPATH/bin 中。

### Go Mod 使用

在一个新的项目中，需要执行`go mod init` 来初始化创建文件`go.mod`，`go.mod` 中会列出所有依赖包的路径和版本。

```
module github.com/xfstart07/watcher

require (
    github.com/apex/log v1.0.0
    github.com/fatih/color v1.7.0 // indirect
    github.com/fsnotify/fsnotify v1.4.7
    github.com/go-ini/ini v1.38.2
    github.com/go-kit/kit v0.7.0
    github.com/go-logfmt/logfmt v0.3.0 // indirect
```

`indirect` 表示这个库是间接引用进来的。

`go mod vendor` 命令可以在项目中创建 `vendor` 文件夹将依赖包拷贝过来。

`go mod download` 命令用于将依赖包缓存到本地Cache起来。