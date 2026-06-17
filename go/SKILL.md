---
name: go
description: Go 后端开发指南，涵盖 API 服务、并发、错误处理与项目结构。在用户编写或修改 Go 服务、handler、repository、中间件时使用。
---

# Go 编程指南

## 核心原则

1. **简洁** — 小函数、直白控制流；能用标准库就不引第三方；避免过早抽象
2. **性能好** — 关注热路径：预分配 slice、减少分配与拷贝、复用连接池；用 `context` 控制超时与取消
3. **扩展性好** — 按层拆分（handler / service / repository）；依赖接口而非具体实现，便于替换与测试

## 项目结构

```
cmd/          # 入口
internal/     # 业务代码（不对外暴露）
pkg/          # 可复用公共包（确有需要时）
```

- 包名简短、职责单一；文件名与主要类型一致
- 配置通过结构体注入，不散落全局变量

## 编码约定

```go
// 错误必须处理或显式返回
if err != nil {
    return fmt.Errorf("fetch user %d: %w", id, err)
}

// 接口由消费方定义，保持精简
type UserStore interface {
    Get(ctx context.Context, id int64) (*User, error)
}
```

- 导出标识符有明确语义；未导出辅助函数放同包
- 用 `errors.Is` / `errors.As` 判断错误类型
- 并发用 `errgroup` 或明确的生命周期管理，避免裸 goroutine 泄漏

## 性能要点

- slice：`make([]T, 0, n)` 已知容量时预分配
- 字符串拼接多段时用 `strings.Builder`
- HTTP / DB 连接复用连接池，设置合理超时
- 避免在循环内重复编译正则、分配大对象
- 需要时再 `go test -bench` 验证，不凭猜测优化

## 扩展要点

- Handler 只做参数解析与响应，业务逻辑进 service
- 数据访问封装在 repository，与存储实现解耦
- 横切关注点（日志、鉴权、限流）用中间件
- 新增能力优先扩展接口与实现，而非修改多处调用

## API 与并发

- 对外 API 带 `context.Context` 作为首参
- 共享数据用 channel 或 sync 原语，明确所有权
- 优雅关闭：监听信号，释放连接与后台任务

## 禁止

- `panic` 处理可预期业务错误（仅用于不可恢复场景）
- 忽略 `error` 或 `_` 吞掉关键失败
- 为简单逻辑引入 ORM、DI 框架等重量级依赖（除非项目已采用）
- 循环依赖包
