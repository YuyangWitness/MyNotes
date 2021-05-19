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

## 函数、方法和接口

- Go程序初始化的时候总是从main.main函数开始，如果在main中导入了其他包，那么会按照顺序进入其他包中，以此执行 `import pacakge -> const -> var -> init()`，在其他包遇到了新的包，也会继续进入新的包。原理特别像洋葱模型，当完全运行完了某个包内代码，则会返回到上个包继续运行。

- 如果在init遇到了goroutinue，不会立刻进入新的线程，只有在进入main.main函数之后才会执行新的goroutinue.

```go
// 具名函数
func Add(a, b int) int {
    return a+b
}

// 匿名函数
var Add = func(a, b int) int {
    return a+b
}
```

```go
func Inc() (v int) {
    defer func(){ v++ } ()
    return 42
}
```

- 匿名函数捕获了外部函数的局部变量`v`，这种函数我们一般叫闭包。闭包对捕获的外部变量并不是传值方式访问，而是以引用的方式访问。
- Go语言的指针地址是有可能会改变的，不是固定的，因为Go语言运行时会自动更新引用了地址变化的栈变量的指针

## 面向并发的内存模型

一个Goroutine会以一个很小的栈启动（可能是2KB或4KB），当遇到深度递归导致当前栈空间不足时，Goroutine会根据需要动态地伸缩栈的大小（主流实现中栈的最大值可达到1GB）。因为启动的代价很小，所以我们可以轻易地启动成千上万个Goroutine。