# 三个项目的协同调整建议

## 调整优先级

### 🔴 必须调整（立即）

#### 1. ai-context 添加 zero-skills 引用

**文件**：`00-instructions.md`

**在 "## File Priority" 部分添加**：

```markdown
## File Priority

1. `workflows.md` - Task patterns
2. `tools.md` - MCP tools
3. `patterns.md` - Code patterns
4. [zero-skills](https://github.com/zeromicro/zero-skills) - Detailed patterns (查阅详细模式)
```

**在 "## Avoid" 之前添加新章节**：

```markdown
## Detailed Patterns

For complete implementation patterns, refer to [zero-skills](https://github.com/zeromicro/zero-skills):

- REST API → [rest-api-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/rest-api-patterns.md)
- RPC Services → [rpc-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/rpc-patterns.md)
- Database → [database-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/database-patterns.md)
- Resilience → [resilience-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/resilience-patterns.md)
- Troubleshooting → [common-issues.md](https://github.com/zeromicro/zero-skills/blob/main/troubleshooting/common-issues.md)
```

#### 2. zero-skills 添加语言切换

**文件**：`README.md`（第一行）

```markdown
# go-zero Skills for AI Agents

English | [简体中文](README_CN.md)
```

#### 3. zero-skills 添加与其他项目的关系说明

**文件**：`README.md`（在 "## For Developers" 之后添加）

```markdown
## Integration with go-zero AI Ecosystem

zero-skills is part of the go-zero AI tools ecosystem:

- **[ai-context](https://github.com/zeromicro/ai-context)** - Concise instructions for GitHub Copilot
- **zero-skills** (this repo) - Detailed knowledge base for all AI assistants
- **[mcp-zero](https://github.com/zeromicro/mcp-zero)** - Runtime tools for Claude Desktop

See [AI Ecosystem Guide](articles/ai-ecosystem-guide.md) for details.
```

### 🟡 应该调整（本周内）

#### 4. mcp-zero 添加 zero-skills 资源支持

**文件**：`main.go`

**添加资源列表函数**：

```go
// 在 main.go 中添加
func getZeroSkillsResources() []map[string]interface{} {
    baseURL := "https://raw.githubusercontent.com/zeromicro/zero-skills/main/"
    
    return []map[string]interface{}{
        {
            "uri":         "zero-skills://patterns/rest-api",
            "name":        "REST API Patterns",
            "description": "Complete REST API patterns with Handler/Logic/Context architecture",
            "mimeType":    "text/markdown",
            "url":         baseURL + "patterns/rest-api-patterns.md",
        },
        {
            "uri":         "zero-skills://patterns/rpc",
            "name":        "RPC Patterns",
            "description": "gRPC service patterns with service discovery and load balancing",
            "mimeType":    "text/markdown",
            "url":         baseURL + "patterns/rpc-patterns.md",
        },
        {
            "uri":         "zero-skills://patterns/database",
            "name":        "Database Patterns",
            "description": "SQL operations, MongoDB, caching, and transactions",
            "mimeType":    "text/markdown",
            "url":         baseURL + "patterns/database-patterns.md",
        },
        {
            "uri":         "zero-skills://patterns/resilience",
            "name":        "Resilience Patterns",
            "description": "Circuit breaker, rate limiting, and load shedding",
            "mimeType":    "text/markdown",
            "url":         baseURL + "patterns/resilience-patterns.md",
        },
        {
            "uri":         "zero-skills://troubleshooting",
            "name":        "Troubleshooting Guide",
            "description": "Common issues and solutions",
            "mimeType":    "text/markdown",
            "url":         baseURL + "troubleshooting/common-issues.md",
        },
    }
}
```

#### 5. mcp-zero README 添加 zero-skills 说明

**文件**：`readme.md`（在 Features 部分之后添加）

```markdown
## Resources

mcp-zero integrates with [zero-skills](https://github.com/zeromicro/zero-skills) to provide:

- 📚 Complete go-zero patterns (REST API, RPC, Database, Resilience)
- ✅ Correct vs ❌ Incorrect examples
- 🔍 Troubleshooting guides
- 🚀 Production best practices

Access these patterns directly through MCP resources when using Claude Desktop.
```

### 🟢 可以调整（本月内）

#### 6. mcp-zero 添加模式搜索工具

**新增工具**：`search_pattern`

```go
{
    Name:        "search_zero_skills",
    Description: "Search zero-skills knowledge base for go-zero patterns and examples",
    InputSchema: map[string]interface{}{
        "type": "object",
        "properties": map[string]interface{}{
            "query": {
                "type":        "string",
                "description": "What to search for (e.g., 'middleware', 'transaction', 'cache')",
            },
            "category": {
                "type":        "string",
                "enum":        []string{"rest", "rpc", "database", "resilience", "troubleshooting", "all"},
                "description": "Category to search in",
                "default":     "all",
            },
        },
        "required": []string{"query"},
    },
}
```

#### 7. zero-skills 添加更多实际案例

**新增目录**：`examples/`

```
examples/
├── rest-api-crud/          # 完整的 CRUD API 示例
├── rpc-service-discovery/  # 带服务发现的 RPC 示例
├── microservices-demo/     # 多服务协作示例
└── production-deployment/  # 生产部署配置
```

#### 8. 统一文档风格

**三个项目都应该**：
- 添加中英文双语支持
- 统一链接格式
- 统一代码示例风格
- 添加版本兼容性说明

## 调整后的协同效果

### 场景 1：开发者使用 GitHub Copilot

```
1. 开发者在 .github/copilot-instructions.md 中配置 ai-context
2. ai-context 引用 zero-skills 获取详细模式
3. Copilot 生成符合 go-zero 规范的代码
```

### 场景 2：开发者使用 Claude Desktop

```
1. 开发者配置 mcp-zero 为 MCP 服务器
2. mcp-zero 提供代码生成工具
3. mcp-zero 同时提供 zero-skills 资源访问
4. Claude 可以：
   - 调用工具生成项目结构
   - 查询 zero-skills 获取实现模式
   - 参考故障排查文档解决问题
```

### 场景 3：开发者使用 Cursor

```
1. 开发者在 Cursor 中添加 ai-context 规则
2. 在提示词中引用 zero-skills URL
3. Cursor AI 获取完整的实现模式
4. 生成符合规范的代码
```

## 实施计划

### Week 1（立即）
- [x] 创建 zero-skills 仓库并发布
- [ ] ai-context 添加 zero-skills 引用
- [ ] zero-skills 添加中文 README
- [ ] 三个项目互相链接

### Week 2
- [ ] mcp-zero 实现 zero-skills 资源访问
- [ ] 测试 Claude Desktop 集成
- [ ] 文档补充使用示例

### Week 3
- [ ] mcp-zero 添加模式搜索工具
- [ ] zero-skills 添加更多示例
- [ ] 收集社区反馈

### Week 4
- [ ] 优化 AI 助手体验
- [ ] 完善文档
- [ ] 发布使用指南

## 成功指标

1. **AI 代码质量**
   - 生成的代码 90%+ 符合 go-zero 规范
   - 减少开发者修改次数

2. **开发效率**
   - 新手从 0 到创建第一个服务时间 < 10 分钟
   - 减少 50%+ 的样板代码编写时间

3. **社区采用**
   - 100+ stars on zero-skills
   - 50+ 个项目使用 ai-context
   - 20+ 个 Claude Desktop 用户使用 mcp-zero

4. **问题解决**
   - 80% 常见问题可通过 troubleshooting 自助解决
   - 减少重复性问题提问

## 维护计划

### 内容同步
- go-zero 新版本发布 → 更新 zero-skills 模式
- zero-skills 更新 → mcp-zero 缓存刷新
- 社区反馈 → 快速迭代改进

### 版本管理
- zero-skills 跟随 go-zero 大版本号
- 支持查询特定版本的模式
- 保持向后兼容

### 社区协作
- 接受社区 PR 贡献新模式
- 定期 review 和更新内容
- 收集使用反馈持续优化
