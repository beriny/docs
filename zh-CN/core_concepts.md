# 核心概念

## 经典 Macaron

为了更快速的启用 Macaron，[`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 提供了一些默认的组件以方便 Web 开发:

```go
m := macaron.Classic()
// ... 可以在这里使用中间件和注册路由
m.Run()
```

下面是 [`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic) 已经包含的功能：

- 请求/响应日志 - [`macaron.Logger`](core_services.md#lu-you-ri-zhi)
- 容错恢复 - [`macaron.Recovery`](core_services.md#rong-cuo-hui-fu)
- 静态文件服务 - [`macaron.Static`](core_services.md#jing-tai-wen-jian)

## Macaron 实例

任何类型为 [`macaron.Macaron`](https://gowalker.org/gopkg.in/macaron.v1#Macaron) 的对象都可以被认为是 Macaron 的实例，您可以在单个程序中使用任意数量的 Macaron 实例。

## 处理器

处理器是 Macaron 的灵魂和核心所在. 一个处理器基本上可以是任何的函数:

```go
m.Get("/", func() string {
	return "hello world"
})
```

如果想要将同一个函数作用于多个路由，则可以使用一个命名函数：

```go
m.Get("/", myHandler)
m.Get("/hello", myHandler)

func myHandler() string {
	return "hello world"
}
```

除此之外，同一个路由还可以注册任意多个处理器：

```go
m.Get("/", myHandler1, myHandler2)

func myHandler1() {
	// ... 处理内容
}

func myHandler2() string {
	return "hello world"
}
```

### 返回值

当一个处理器返回结果的时候, Macaron 将会把返回值作为字符串写入到当前的 [`http.ResponseWriter`](http://gowalker.org/net/http#ResponseWriter) 里面：

```go
m.Get("/", func() string {
	return "hello world" // HTTP 200 : "hello world"
})

m.Get("/", func() *string {
	str := "hello world"
	return &str // HTTP 200 : "hello world"
})

m.Get("/", func() []byte {
    return []byte("hello world") // HTTP 200 : "hello world"
})

m.Get("/", func() error {
	// 返回 nil 则什么都不会发生
	return nil 
}, func() error {
	// ... 得到了错误
	return err // HTTP 500 : <错误消息>
})
```

另外你也可以选择性的返回状态码（仅适用于 `string` 和 `[]byte` 类型）:

```go
m.Get("/", func() (int, string) {
	return 418, "i'm a teapot" // HTTP 418 : "i'm a teapot"
})

m.Get("/", func() (int, *string) {
	str := "i'm a teapot"
	return 418, &str // HTTP 418 : "i'm a teapot"
})

m.Get("/", func() (int, []byte) {
	return 418, []byte("i'm a teapot") // HTTP 418 : "i'm a teapot"
})
```

### 服务注入

处理器是通过反射来调用的，Macaron 通过 [依赖注入](http://baike.baidu.com/view/1486379.htm?from_id=5177233&type=syn&fromtitle=%E4%BE%9D%E8%B5%96%E6%B3%A8%E5%85%A5&fr=aladdin) 来为处理器注入参数列表。 **这样使得 Macaron 与 Go 语言的 [`http.HandlerFunc`](https://gowalker.org/net/http#HandlerFunc) 接口完全兼容**。

如果你加入一个参数到你的处理器, Macaron 将会搜索它参数列表中的服务，并且通过类型判断来解决依赖关系：

```go
m.Get("/", func(resp http.ResponseWriter, req *http.Request) {
	// resp 和 req 是由 Macaron 默认注入的服务
	resp.WriteHeader(200) // HTTP 200
})
```

在您的代码中最常用的服务应该是 [`*macaron.Context`](core_services.md#qing-qiu-shang-xia-wen-context)：

```go
m.Get("/", func(ctx *macaron.Context) {
	ctx.Resp.WriteHeader(200) // HTTP 200
})
```

下面的这些服务已经被包含在经典 Macaron 中（[`macaron.Classic`](https://gowalker.org/gopkg.in/macaron.v1#Classic)）：

- [`*macaron.Context`](core_services.md#qing-qiu-shang-xia-wen-context) - HTTP 请求上下文
- [`*log.Logger`](core_services.md#quan-ju-ri-zhi) - Macaron 全局日志器
- [`http.ResponseWriter`](core_services.md#xiang-ying-liu) - HTTP 响应流
- [`*http.Request`](core_services.md#qing-qiu-dui-xiang) - HTTP 请求对象

### 中间件机制

中间件处理器是工作于请求和路由之间的。本质上来说和 Macaron 其他的处理器没有分别. 您可以使用如下方法来添加一个中间件处理器到队列中:

```go
m.Use(func() {
  // 处理中间件事务
})
```

你可以通过 `Handlers` 函数对中间件队列实现完全的控制. 它将会替换掉之前的任何设置过的处理器:

```go
m.Handlers(
	Middleware1,
	Middleware2,
	Middleware3,
)
```

中间件处理器可以非常好处理一些功能，包括日志记录、授权认证、会话（sessions）处理、错误反馈等其他任何需要在发生在 HTTP 请求之前或者之后的操作:

```go
// 验证一个 API 密钥
m.Use(func(ctx *macaron.Context) {
	if ctx.Req.Header.Get("X-API-KEY") != "secret123" {
		ctx.Resp.WriteHeader(http.StatusUnauthorized)
	}
})
```

## Macaron 环境变量

一些 Macaron 处理器依赖 `macaron.Env` 全局变量为开发模式和部署模式表现出不同的行为，不过更建议使用环境变量 `MACARON_ENV=production` 来指示当前的模式为部署模式。

## 处理器工作流

![](../images/handler_workflow.png)
