# Command System Architecture

> **Reference**: Main diagram in [ARCHITECTURE.md](../ARCHITECTURE.md)

## Overview

The Command System provides ~85 slash commands for common operations like commit, review, and configuration.

## Command Categories

```mermaid
flowchart TB
    subgraph Input["User Input"]
        Prompt["Natural Prompt"]
        Slash["Slash Command\n/cmd"]
    end

    subgraph Parse["Command Parsing"]
        Detect["Detect Slash"]
        Match["Match Command\nby name"]
        Validate["Validate Args"]
    end

    subgraph Execute["Command Execution"]
        PromptCmd["PromptCommand\n→ LLM"]
        LocalCmd["LocalCommand\n→ Text"]
        JSXCmd["LocalJSXCommand\n→ React JSX"]
    end

    subgraph Output["Output"]
        Stream["Stream Response"]
        Render["Render Output"]
    end

    Input --> Parse
    Parse --> Execute
    Execute --> Output
```

## Command Types

| Type | Interface | Behavior | Examples |
|------|-----------|----------|----------|
| `PromptCommand` | Returns prompt text | Formats prompt, sends to LLM | `/commit`, `/review`, `/compact` |
| `LocalCommand` | Text output | Runs in-process, outputs text | `/cost`, `/version`, `/context` |
| `LocalJSXCommand` | React JSX output | Runs in-process, outputs React | `/doctor`, `/config`, `/resume` |

## Key Commands

| Category | Commands |
|----------|----------|
| **Git** | `/commit`, `/review`, `/diff`, `/compact` |
| **Info** | `/cost`, `/version`, `/context`, `/status` |
| **Diagnostics** | `/doctor`, `/health` |
| **Config** | `/config`, `/settings` |
| **Sessions** | `/resume`, `/clear`, `/sessions` |
| **MCP** | `/mcp add`, `/mcp list`, `/mcp remove` |
| **Plugins** | `/plugins install`, `/plugins list` |
| **Tasks** | `/tasks`, `/task create` |

## Command Execution Flow

```mermaid
sequenceDiagram
    participant U as User
    participant P as PromptInput
    participant C as Command System
    participant Q as Query Engine

    U->>P: /commit
    P->>C: detect slash command
    C->>C: parse command + args
    alt PromptCommand
        C->>C: format prompt
        C->>Q: send to LLM
    else LocalCommand
        C->>C: execute locally
        C-->>P: return text
    else JSXCommand
        C->>C: execute locally
        C-->>P: return JSX
    end
    Q-->>P: stream response
    P-->>U: display output
```

## Key Files

| Component | File | Description |
|-----------|------|-------------|
| Command Registry | `src/commands.ts` | Command definitions (~85 commands) |
| Command Hooks | `src/hooks/useCommands.ts` | Command hooks |
| Bridge Safe Check | `src/commands.ts:isBridgeSafeCommand` | IDE bridge validation |

---

*See also: [ARCHITECTURE.md](../ARCHITECTURE.md), [query-engine.md](query-engine.md)*