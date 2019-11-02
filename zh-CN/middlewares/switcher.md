# 服务多个站点

辅助模块 switcher 为您的应用提供多个 [Macaron 实例](../core_concepts.md#macaron-shi-li) 的支持。

- [GitHub](https://github.com/go-macaron/switcher)
- [API 文档](https://gowalker.org/github.com/go-macaron/switcher)

## 下载安装

```sh
go get github.com/go-macaron/switcher
```
	
## 使用示例

如果您想要运行 2 个或 2 个以上的 [Macaron 实例](../core_concepts.md#macaron-shi-li) 在一个程序中，该辅助模块便可为此类需求提供便利：

```go
func main() {
	m1 := macaron.Classic()
	// 注册 m1 实例的中间件和路由

	m2 := macaron.Classic()
	// 注册 m2 实例的中间件和路由

	hs := switcher.NewHostSwitcher()
	// 设置实例所对应的主机地址
	hs.Set("gowalker.org", m1)
	hs.Set("gogs.io", m2)
	hs.Run()
}
```

默认情况下，即 `macaron.DEV` 模式，出于对调试的便利性，该程序会监听多个端口，包括 `4000`（用于实例 `m1`）和 `4001`（用于实例 `m2`）。而当模式为 `macaron.PROD` 时，则只会监听一个端口，即 `4000`。

### 动态匹配

如果您有多个子域名需要使用一个 Macaron 实例来处理，则可以通过以下方式来动态匹配：

```go
// ...
m := macaron.Classic()
// 注册 m 实例的中间件和路由

hs := macaron.NewHostSwitcher()
// 设置实例所对应的主机地址
hs.Set("*.example.com", m)
hs.Run()
// ...
```