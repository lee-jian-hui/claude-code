# Tool System Architecture

> **Reference**: Main diagram in [ARCHITECTURE.md](../ARCHITECTURE.md)

## Overview

The Tool System provides ~40+ built-in tools for file operations, execution, web access, and more.

## Tool Categories

```mermaid
flowchart TB
    subgraph Tools["Tool System (~40+ Tools)"]
        FileIO["File I/O\nRead · Write · Edit · Glob · Grep"]
        Exec["Execution\nBash · PowerShell · REPL"]
        Agent["Agent Orchestration\nAgentTool · SendMessage · Worktree"]
        Web["Web\nWebFetch · WebSearch"]
        Task["Task Management\nCreate · Update · Stop · List"]
        MCP["MCP Integration\nMCPTool · ToolSearch · Resources"]
        LSP["Language Server\nLSP hover · definition"]
        Skill["Skills\nSkillTool · User Scripts"]
    end

    subgraph Execution["Tool Execution"]
        Perm["Permission Check"]
        Validate["Input Validation"]
        Execute["Tool.call()"]
        Result["Tool Result"]
    end

    subgraph Output["Output Processing"]
        Truncate["Result Truncation"]
        Render["Render Messages"]
        Stream["Stream to UI"]
    end

    Tools --> Execution
    Execution --> Output

    subgraph Backend["External Systems"]
        FS["Filesystem"]
        Shell["Shell / OS"]
        MCPSrv["MCP Servers"]
        LSPSrv["Language Servers"]
        WebExt["Web (HTTP)"]
    end

    Execution --> Backend
```

## Tool Interface

```typescript
type Tool<Input, Output, P> = {
  name: string
  aliases?: string[]
  inputSchema: Input  // Zod schema
  
  call(args, context, canUseTool, parentMessage, onProgress): Promise<ToolResult<Output>>
  
  // Permissions
  isConcurrencySafe(input): boolean
  isReadOnly(input): boolean
  isDestructive?(input): boolean
  checkPermissions(input, context): Promise<PermissionResult>
  
  // UI
  description(input, options): Promise<string>
  renderToolUseMessage()
  renderToolResultMessage()
}
```

## Tool Categories Detail

| Category | Tools | Purpose |
|----------|-------|---------|
| **File I/O** | FileReadTool, FileWriteTool, FileEditTool, GlobTool, GrepTool | File operations |
| **Execution** | BashTool, PowerShellTool, REPLTool | Shell/command execution |
| **Agent** | AgentTool, SendMessageTool, EnterPlanModeTool, EnterWorktreeTool | Multi-agent coordination |
| **Web** | WebFetchTool, WebSearchTool | Web access |
| **Task** | TaskCreateTool, TaskGetTool, TaskUpdateTool, TaskStopTool, TaskListTool | Background tasks |
| **MCP** | MCPTool, ToolSearchTool, ListMcpResourcesTool, ReadMcpResourceTool | External tool servers |
| **LSP** | LSPTool | Language server operations |
| **Skills** | SkillTool | User-defined scripts |

## Tool Execution Flow

```mermaid
sequenceDiagram
    participant Q as Query Engine
    participant P as Permission Check
    participant T as Tool
    participant S as Sanitization
    participant R as Result

    Q->>P: tool_use block detected
    P->>S: validate input
    S-->>P: sanitized input
    P->>P: check allow/deny/ask rules
    alt allow or user approved
        P->>T: execute tool
        T->>R: return result
        R-->>Q: tool_result message
    else denied
        P-->>Q: permission denied
    end
```

## Key Files

| Component | File | Description |
|-----------|------|-------------|
| Tool Interface | `src/Tool.ts` | Tool contract (~794 lines) |
| Tool Registry | `src/tools.ts` | `getAllBaseTools()` |
| Execution | `src/tools/StreamingToolExecutor.ts` | Streaming tool executor |

---

*See also: [ARCHITECTURE.md](../ARCHITECTURE.md), [security.md](security.md)*