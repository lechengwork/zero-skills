# go-zero Skills - AI 助手的知识库

[English](README.md) | 简体中文

这个仓库包含为 AI 编程助手（Claude、GitHub Copilot、Cursor 等）优化的 go-zero 框架知识和模式，帮助开发者更高效地使用 go-zero 构建微服务应用。

## 目标

为 AI 助手提供结构化的 go-zero 知识，使其能够：
- 生成符合 go-zero 规范的准确代码
- 提供上下文相关的建议和补全
- 帮助开发者遵循最佳实践
- 快速解决常见问题

## 目录结构

```
zero-skills/
├── getting-started/     # 快速开始指南
├── patterns/            # 框架特定的模式和约定
│   ├── rest-api-patterns.md      # REST API 完整模式
│   ├── rpc-patterns.md           # RPC 服务模式
│   ├── database-patterns.md      # 数据库操作模式
│   └── resilience-patterns.md    # 弹性保护模式
├── best-practices/      # 生产级最佳实践
├── troubleshooting/     # 常见问题和解决方案
└── articles/            # 深度文章和教程
```

## 给 AI 助手的说明

在协助 go-zero 开发时：
1. 从 `patterns/` 开始了解框架约定
2. 参考 `examples/` 获取完整的工作代码
3. 遵循 `best-practices/` 生成生产级代码
4. 使用 `troubleshooting/` 解决错误

## 给开发者的说明

### 与 GitHub Copilot 配合使用

在项目的 `.github/` 目录中引用这个仓库：

```markdown
## go-zero 开发指南

参考 zero-skills 仓库的详细模式：
- REST API: https://github.com/zeromicro/zero-skills/blob/main/patterns/rest-api-patterns.md
- RPC 服务: https://github.com/zeromicro/zero-skills/blob/main/patterns/rpc-patterns.md
```

### 与 Claude Desktop 配合使用

配合 [mcp-zero](https://github.com/zeromicro/mcp-zero) 使用，Claude 可以直接访问这些模式。

### 与 Cursor 配合使用

在 Cursor 的项目规则中引用 zero-skills 的 URL，让 AI 了解 go-zero 最佳实践。

## 特色功能

### ✅ 正确做法 vs ❌ 错误做法

每个模式都包含对比示例：

```go
// ✅ 正确：Handler 只处理 HTTP 相关逻辑
func (h *Handler) Handle(w http.ResponseWriter, r *http.Request) {
    var req types.Request
    if err := httpx.Parse(r, &req); err != nil {
        httpx.ErrorCtx(r.Context(), w, err)
        return
    }
    
    l := logic.NewLogic(r.Context(), h.svcCtx)
    resp, err := l.Process(&req)
    // ...
}

// ❌ 错误：不要在 Handler 中写业务逻辑
func (h *Handler) Handle(w http.ResponseWriter, r *http.Request) {
    // 直接查询数据库、处理业务逻辑等
    user, _ := h.svcCtx.UserModel.FindOne(ctx, id)
    // ...
}
```

### 📚 完整的代码示例

不是代码片段，而是可以直接运行的完整示例，包括：
- 完整的类型定义
- 错误处理
- 配置示例
- 测试代码

### 🔍 详细的故障排查

常见问题和解决方案，包括：
- 症状描述
- 根本原因
- 完整的解决步骤
- 预防措施

## 与 go-zero AI 生态的关系

zero-skills 是 go-zero AI 工具生态的一部分：

```
ai-context        → 简明的工作指令（给 GitHub Copilot）
zero-skills       → 详细的知识库（给所有 AI 助手）
mcp-zero          → 运行时工具调用（给 Claude Desktop）
```

详细说明参见：[go-zero AI 工具生态指南](articles/ai-ecosystem-guide.md)

## 内容示例

### REST API 模式

```go
// Handler 层 - 只处理 HTTP
func CreateUserHandler(svcCtx *svc.ServiceContext) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        var req types.CreateUserRequest
        if err := httpx.Parse(r, &req); err != nil {
            httpx.ErrorCtx(r.Context(), w, err)
            return
        }

        l := logic.NewCreateUserLogic(r.Context(), svcCtx)
        resp, err := l.CreateUser(&req)
        if err != nil {
            httpx.ErrorCtx(r.Context(), w, err)
        } else {
            httpx.OkJsonCtx(r.Context(), w, resp)
        }
    }
}

// Logic 层 - 业务逻辑实现
func (l *CreateUserLogic) CreateUser(req *types.CreateUserRequest) (*types.CreateUserResponse, error) {
    // 验证
    if err := l.validateUser(req); err != nil {
        return nil, err
    }

    // 业务逻辑
    user := &model.User{
        Name:  req.Name,
        Email: req.Email,
    }

    // 数据库操作
    result, err := l.svcCtx.UserModel.Insert(l.ctx, user)
    if err != nil {
        return nil, err
    }

    userId, _ := result.LastInsertId()
    return &types.CreateUserResponse{Id: userId}, nil
}
```

## 贡献指南

欢迎贡献！请确保：
- 示例完整且经过测试
- 模式遵循官方 go-zero 约定
- 内容结构化，便于 AI 理解
- 包含正确和错误的示例对比

### 贡献内容

1. Fork 本仓库
2. 创建特性分支：`git checkout -b feature/new-pattern`
3. 提交更改：`git commit -am 'Add new pattern for XXX'`
4. 推送分支：`git push origin feature/new-pattern`
5. 提交 Pull Request

### 内容要求

- 使用清晰的标题和章节
- 提供完整的代码示例
- 说明使用场景
- 包含配置示例
- 添加故障排查提示

## 版本说明

当前版本：**v1.0.0**

兼容 go-zero 版本：
- v1.6.x ✅
- v1.5.x ✅

## 许可证

MIT License - 与 go-zero 框架相同

## 相关链接

- [go-zero 框架](https://github.com/zeromicro/go-zero)
- [go-zero 文档](https://go-zero.dev)
- [ai-context](https://github.com/zeromicro/ai-context) - GitHub Copilot 指令
- [mcp-zero](https://github.com/zeromicro/mcp-zero) - MCP 工具服务器

## 社区

- Discord: https://discord.gg/4JQvC5A4Fe
- 微信群：加入 go-zero 开发者社区
- GitHub Discussions: https://github.com/zeromicro/go-zero/discussions
