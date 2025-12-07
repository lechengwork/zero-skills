# Integration with go-zero AI Ecosystem

This document describes how `zero-skills` integrates with other go-zero AI tools.

## Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    AI Coding Assistant                       │
│              (Claude, Copilot, Cursor, etc.)                 │
└───────────────────┬─────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
        ▼                       ▼
┌──────────────┐      ┌─────────────────┐
│  ai-context  │      │    mcp-zero     │
│   (GitHub)   │      │  (MCP Server)   │
│              │      │                 │
│ High-level   │      │ Runtime query   │
│ principles   │      │ interface       │
└──────┬───────┘      └────────┬────────┘
       │                       │
       │                       │
       └───────────┬───────────┘
                   │
                   ▼
          ┌────────────────┐
          │  zero-skills   │
          │                │
          │ Detailed       │
          │ patterns &     │
          │ examples       │
          └────────────────┘
```

## 1. Integration with ai-context

### Current State
[ai-context](https://github.com/zeromicro/ai-context) provides `.github/copilot-instructions.md` that gives AI assistants:
- Project architecture overview
- Key conventions
- Coding standards
- Build commands

### Integration Approach

**Reference zero-skills for detailed patterns:**

Add to `.github/copilot-instructions.md`:

```markdown
## Detailed Implementation Patterns

For complete, working examples and detailed patterns, refer to [zero-skills](https://github.com/zeromicro/zero-skills):

### When to Reference:
- **REST API development** → [rest-api-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/rest-api-patterns.md)
- **RPC service development** → [rpc-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/rpc-patterns.md)
- **Database operations** → [database-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/database-patterns.md)
- **Production resilience** → [resilience-patterns.md](https://github.com/zeromicro/zero-skills/blob/main/patterns/resilience-patterns.md)
- **Troubleshooting** → [common-issues.md](https://github.com/zeromicro/zero-skills/blob/main/troubleshooting/common-issues.md)

### Pattern Format:
Each pattern includes:
- ✅ Correct implementation with complete code
- ❌ Common mistakes to avoid
- Configuration examples
- When to use the pattern
```

**Benefits:**
- `ai-context` stays concise (principles)
- `zero-skills` provides depth (implementation)
- AI assistants get layered understanding

## 2. Integration with mcp-zero

### Current State
[mcp-zero](https://github.com/zeromicro/mcp-zero) provides an MCP (Model Context Protocol) server that allows AI assistants to:
- Query go-zero documentation
- Get code samples
- Access best practices

### Proposed Integration

**Serve zero-skills content through mcp-zero:**

#### Option A: Add zero-skills as MCP Resources

```go
// In mcp-zero server
func (s *Server) ListResources() []Resource {
    return []Resource{
        // Existing resources...

        // Add zero-skills patterns
        {
            URI:         "zero-skills://patterns/rest-api",
            Name:        "REST API Patterns",
            Description: "Complete REST API implementation patterns",
            MimeType:    "text/markdown",
        },
        {
            URI:         "zero-skills://patterns/rpc",
            Name:        "RPC Patterns",
            Description: "Complete RPC service patterns",
            MimeType:    "text/markdown",
        },
        {
            URI:         "zero-skills://patterns/database",
            Name:        "Database Patterns",
            Description: "SQL, MongoDB, and caching patterns",
            MimeType:    "text/markdown",
        },
        {
            URI:         "zero-skills://patterns/resilience",
            Name:        "Resilience Patterns",
            Description: "Circuit breaker, rate limiting, load shedding",
            MimeType:    "text/markdown",
        },
        {
            URI:         "zero-skills://troubleshooting",
            Name:        "Common Issues",
            Description: "Troubleshooting guide for common problems",
            MimeType:    "text/markdown",
        },
    }
}

func (s *Server) ReadResource(uri string) (string, error) {
    // Fetch from zero-skills repository or local cache
    switch uri {
    case "zero-skills://patterns/rest-api":
        return fetchPattern("rest-api-patterns.md")
    case "zero-skills://patterns/rpc":
        return fetchPattern("rpc-patterns.md")
    // ... other patterns
    }
}
```

#### Option B: Add zero-skills as MCP Tools

```go
// Add tools for querying patterns
func (s *Server) ListTools() []Tool {
    return []Tool{
        {
            Name:        "search_pattern",
            Description: "Search zero-skills for specific implementation patterns",
            InputSchema: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "query": {
                        "type":        "string",
                        "description": "Pattern to search (e.g., 'REST handler', 'RPC client', 'caching')",
                    },
                    "category": {
                        "type":        "string",
                        "enum":        []string{"rest", "rpc", "database", "resilience", "all"},
                        "description": "Pattern category to search in",
                    },
                },
                "required": []string{"query"},
            },
        },
        {
            Name:        "get_example",
            Description: "Get complete working example from zero-skills",
            InputSchema: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "example_type": {
                        "type": "string",
                        "enum": []string{
                            "rest_handler",
                            "rpc_service",
                            "database_transaction",
                            "caching",
                            "middleware",
                            "circuit_breaker",
                        },
                        "description": "Type of example to retrieve",
                    },
                },
                "required": []string{"example_type"},
            },
        },
        {
            Name:        "troubleshoot",
            Description: "Get solutions for common go-zero issues",
            InputSchema: map[string]interface{}{
                "type": "object",
                "properties": map[string]interface{}{
                    "problem": {
                        "type":        "string",
                        "description": "Description of the problem",
                    },
                    "error_message": {
                        "type":        "string",
                        "description": "Error message if available",
                    },
                },
                "required": []string{"problem"},
            },
        },
    }
}
```

#### Option C: Hybrid Approach (Recommended)

Combine both resources and tools:
- **Resources** for browsing complete patterns
- **Tools** for targeted queries and search

### Implementation Strategy

1. **Phase 1**: Add zero-skills as static resources
   - Embed markdown files in mcp-zero binary
   - Serve via MCP resource protocol
   - Quick to implement, low latency

2. **Phase 2**: Add intelligent query tools
   - Implement semantic search over zero-skills content
   - Add contextual example retrieval
   - Enable troubleshooting assistant

3. **Phase 3**: Dynamic updates
   - Fetch from zero-skills repository at runtime
   - Cache with TTL for performance
   - Support version selection

### Example Usage Flow

```typescript
// AI Assistant using mcp-zero with zero-skills integration

// 1. User asks: "Create a REST API endpoint with authentication"

// AI Assistant queries mcp-zero:
const pattern = await mcp.callTool("search_pattern", {
  query: "REST API handler authentication",
  category: "rest"
});

// Returns relevant section from rest-api-patterns.md with:
// - Handler pattern
// - Middleware pattern
// - Complete working example

// 2. User encounters error: "404 not found"

const solution = await mcp.callTool("troubleshoot", {
  problem: "API endpoint returns 404",
  error_message: "404 page not found"
});

// Returns relevant section from common-issues.md with:
// - Cause: Route registration or prefix issue
// - Solution: Check API definition prefix
// - Complete example
```

## 3. Unified Workflow

### For AI Assistants

```
1. Load ai-context (copilot-instructions.md)
   → Understand go-zero principles & conventions

2. Connect to mcp-zero MCP server
   → Access runtime documentation & tools

3. Query zero-skills via mcp-zero
   → Get specific implementation patterns

4. Generate code following patterns
   → Produce accurate, idiomatic go-zero code
```

### For Developers

```
1. Add .github/copilot-instructions.md (from ai-context)
   → Enable GitHub Copilot integration

2. Configure MCP client (Claude Desktop, etc.)
   → Point to mcp-zero server
   → Enable zero-skills resources

3. Start coding
   → AI assistant has full context
   → Gets accurate patterns & examples
   → Produces production-ready code
```

## 4. Content Organization

### ai-context
**Purpose:** High-level project guidance
**Format:** Single copilot-instructions.md file
**Content:**
- Project overview
- Architecture principles
- Key conventions
- Build/test commands

**Update Frequency:** When architecture changes

### zero-skills
**Purpose:** Detailed implementation patterns
**Format:** Structured markdown files
**Content:**
- Complete code examples
- Correct/incorrect patterns
- Configuration examples
- Troubleshooting guides

**Update Frequency:** When patterns evolve or new features added

### mcp-zero
**Purpose:** Runtime query interface
**Format:** MCP server (Go)
**Content:**
- Serves go-zero docs
- Provides code samples
- Integrates zero-skills patterns
- Enables semantic search

**Update Frequency:** Continuous (serves latest content)

## 5. Benefits of Integration

### For AI Assistants
- **Layered understanding**: Principles → Patterns → Implementation
- **Accurate code generation**: Complete, tested examples
- **Context-aware**: Right pattern for the situation
- **Self-correcting**: Troubleshooting guide for errors

### For Developers
- **Faster onboarding**: AI generates correct code from day 1
- **Consistent code**: All AI-generated code follows same patterns
- **Less debugging**: Patterns include common mistakes to avoid
- **Better learning**: Examples show both right and wrong approaches

### For go-zero Ecosystem
- **Lower barrier to entry**: New users get AI assistance
- **Higher code quality**: AI follows best practices
- **Better documentation**: Living examples vs static docs
- **Community growth**: Easier to contribute patterns

## 6. Maintenance Strategy

### Keeping Content Synchronized

```
go-zero (framework)
    ↓ (changes)
ai-context (principles)
    ↓ (update instructions)
zero-skills (patterns)
    ↓ (update examples)
mcp-zero (serves content)
    ↓ (fetches latest)
AI Assistants (generate code)
```

### Update Triggers

1. **Major framework changes** → Update all three
2. **New features** → Add patterns to zero-skills
3. **Best practice evolution** → Update zero-skills examples
4. **Common issues identified** → Add to troubleshooting

### Version Alignment

```yaml
# In zero-skills repository
version: 1.6.0  # Aligned with go-zero version
compatible_versions:
  - "1.6.x"
  - "1.5.x"
last_updated: 2025-12-07
```

## 7. Implementation Roadmap

### Immediate (Week 1)
- [x] Create zero-skills repository with initial patterns
- [ ] Add reference to zero-skills in ai-context
- [ ] Document integration approach

### Short-term (Month 1)
- [ ] Implement zero-skills resources in mcp-zero
- [ ] Add basic search/query tools
- [ ] Test with Claude Desktop + MCP

### Medium-term (Quarter 1)
- [ ] Add semantic search over patterns
- [ ] Implement contextual example retrieval
- [ ] Add interactive troubleshooting

### Long-term (Ongoing)
- [ ] Community contributions to patterns
- [ ] Expand example coverage
- [ ] Multi-language support (Chinese + English)
- [ ] Version-specific pattern serving

## 8. Call to Action

### For ai-context maintainers
Add reference to zero-skills in copilot-instructions.md

### For mcp-zero maintainers
Implement zero-skills integration:
1. Add as MCP resources
2. Implement search tools
3. Add troubleshooting assistant

### For zero-skills maintainers
1. Keep patterns synchronized with go-zero releases
2. Add examples from real-world usage
3. Expand troubleshooting guide
4. Accept community contributions

## Conclusion

The three repositories work together to provide comprehensive AI assistance:

- **ai-context**: "What is go-zero?" (principles)
- **zero-skills**: "How do I use go-zero?" (patterns)
- **mcp-zero**: "Give me go-zero help" (interface)

Together, they enable AI assistants to generate accurate, production-ready go-zero code while helping developers learn the framework faster.
