# Query Engine Architecture

> **Reference**: Main diagram in [ARCHITECTURE.md](../ARCHITECTURE.md)

## Overview

The Query Engine handles the core conversation loop - building prompts, calling the Anthropic API, processing streams, and executing tools.

## Detailed Flow Diagram

```mermaid
flowchart TB
    subgraph Input["Input Processing"]
        Prompt["User Prompt"]
        SlashCmd["Slash Command"]
        Parse["Parse Input"]
    end

    subgraph PromptBuild["System Prompt Builder"]
        Base["Base System Prompt"]
        Memory["CLAUDE.md Memory"]
        Git["Git Context\n(branch, status, diff)"]
        Files["Recent Files"]
        ToolsDesc["Tool Descriptions"]
        Combine["Combine All"]
    end

    subgraph APICall["Anthropic API"]
        BuildReq["Build Request"]
        Stream["Streaming\nPOST /messages"]
        Model["Model Selection"]
        Budget["Token Budget"]
    end

    subgraph StreamProc["Stream Processing"]
        CBStart["content_block_start"]
        CBDelta["content_block_delta"]
        CBStop["content_block_stop"]
        ToolUse["tool_use Block"]
        TextResp["Text Response"]
        Thinking["Thinking Block"]
    end

    subgraph ToolExec["Tool Execution"]
        PermCheck["Permission Check\n(useCanUseTool)"]
        RuleMatch["Rule Matching\n(allow/deny/ask)"]
        Dangerous["Dangerous Pattern\nDetection"]
        UserPrompt["User Prompt"]
        Execute["Tool.call()"]
        Result["Tool Result"]
    end

    subgraph Loop["Continue Loop"]
        Append["Append tool_result"]
        Recurse["Re-call API"]
        CheckStop["Check stop_reason"]
    end

    subgraph States["Query States"]
        Completed["completed"]
        Blocking["blocking_limit"]
        MaxTurns["max_turns"]
        ToolUseState["tool_use"]
        CompactRetry["reactive_compact_retry"]
        TokenRec["max_output_tokens_recovery"]
        BudgetCont["token_budget_continuation"]
    end

    %% Input to Prompt
    Prompt --> Parse
    SlashCmd --> Parse
    Parse -->|"slash"| PromptBuild
    Parse -->|"natural"| PromptBuild

    %% Prompt Build
    Base --> Combine
    Memory --> Combine
    Git --> Combine
    Files --> Combine
    ToolsDesc --> Combine
    Combine --> BuildReq

    %% API Call
    BuildReq --> Model
    BuildReq --> Budget
    Model --> Stream
    Budget --> Stream

    %% Stream Processing
    Stream --> CBStart
    CBStart --> CBDelta
    CBDelta --> CBStop
    CBStop --> ToolUse
    CBStop --> TextResp
    CBDelta --> Thinking

    %% Tool Execution
    ToolUse --> PermCheck
    PermCheck --> RuleMatch
    RuleMatch -->|"deny"| Blocked[Blocked]
    RuleMatch --> Dangerous
    Dangerous -->|"match"| UserPrompt
    Dangerous -->|"safe"| Execute
    UserPrompt --> Execute
    Execute --> Result

    %% Continue Loop
    Result --> Append
    Append --> Recurse
    Recurse --> CheckStop
    CheckStop -->|"end_turn"| Completed
    CheckStop -->|"tool_use"| ToolUseState
    CheckStop -->|"continuation"| BudgetCont

    %% Edge Cases
    ToolUseState --> CompactRetry
    CompactRetry --> Recurse
    TokenRec --> Recurse

    %% Styling
    style Input fill:#e3f2fd,stroke:#1976d2
    style PromptBuild fill:#e8f5e9,stroke:#388e3c
    style APICall fill:#fff3e0,stroke:#f57c00
    style StreamProc fill:#f3e5f5,stroke:#7b1fa2
    style ToolExec fill:#ffebee,stroke:#d32f2f
    style Loop fill:#e0f7fa,stroke:#0097a7
    style States fill:#fce4ec,stroke:#c2185b
```

## Query Loop States

| State | Description | Action |
|-------|-------------|--------|
| `completed` | Normal termination | End conversation |
| `blocking_limit` | Rate/compute limits | Show error |
| `max_turns` | Exceeded iterations | Stop loop |
| `tool_use` | Execute tool | Continue loop |
| `reactive_compact_retry` | Compress context | Retry |
| `max_output_tokens_recovery` | Token limit hit | Recover |
| `token_budget_continuation` | Budget exhausted | Continue |

## Key Files

| Component | File | Description |
|-----------|------|-------------|
| Query Engine | `src/QueryEngine.ts` | Core query class |
| Query Loop | `src/query.ts` | Async generator loop |
| Query State | `src/query/transitions.ts` | State machine |
| Tool Executor | `src/tools/StreamingToolExecutor.ts` | Tool execution |

## Edge Case Handling

1. **Auto-compact**: Compress context when approaching token limits
2. **Token budget**: Track and enforce output token limits
3. **Max output tokens**: Recovery when API returns limit error
4. **Reactive compact**: Retry after context overflow

---

*See also: [ARCHITECTURE.md](../ARCHITECTURE.md), [security.md](security.md)*