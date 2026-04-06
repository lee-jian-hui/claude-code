# Startup Architecture

> **Reference**: Main diagram in [ARCHITECTURE.md](../ARCHITECTURE.md)

## Overview

The startup flow handles CLI initialization, fast paths, parallel prefetching, and full application initialization.

## Detailed Flow Diagram

```mermaid
flowchart TB
    subgraph CLI["CLI Entry cli.tsx"]
        Args["process.argv"]
        FastPath["Fast Path Check"]
        Version["--version / -v"]
        DumpPrompt["--dump-system-prompt"]
        ChromeMCP["--claude-in-chrome-mcp"]
        ChromeHost["--chrome-native-host"]
        CompMCP["--computer-use-mcp"]
        DaemonWorker["--daemon-worker"]
        Bridge["bridge / remote"]
        Daemon["daemon"]
        SubCmds["ps / logs / attach / kill / new / list / reply"]
        FullLoad["Full CLI Load"]
    end

    subgraph Main["main.tsx"]
        Prefetch["Parallel Prefetch\n~200ms savings"]
        MDM["MDM Settings\n(plutil/reg query)"]
        Keychain["Keychain\n(OAuth + API key)"]
        APIConn["API Preconnect\n(TCP+TLS warmup)"]
        Imports["Heavy Imports"]
    end

    subgraph Init["init() Initialization"]
        EnableConfig["enableConfigs()"]
        OAuth["OAuth Setup"]
        Config["Config Loading"]
        MCP["MCP Servers"]
        Plugins["Plugins"]
        Skills["Skills"]
        GrowthBook["GrowthBook\nFeature Flags"]
    end

    subgraph REPL["REPL Launch"]
        Launch["launchRepl()"]
        State["AppStateStore"]
        UI["React + Ink UI"]
    end

    %% CLI Flow
    Args --> FastPath
    FastPath -->|"version"| Version
    FastPath -->|"dump-system-prompt"| DumpPrompt
    FastPath -->|"claude-in-chrome-mcp"| ChromeMCP
    FastPath -->|"chrome-native-host"| ChromeHost
    FastPath -->|"computer-use-mcp"| CompMCP
    FastPath -->|"daemon-worker"| DaemonWorker
    FastPath -->|"bridge/remote"| Bridge
    FastPath -->|"daemon"| Daemon
    FastPath -->|"subcommand"| SubCmds
    FastPath -->|"other"| FullLoad
    
    %% Main Flow
    FullLoad --> Prefetch
    Prefetch -->|"parallel"| MDM
    Prefetch -->|"parallel"| Keychain
    Prefetch -->|"parallel"| APIConn
    Prefetch --> Imports
    
    %% Init Flow
    Imports --> EnableConfig
    EnableConfig --> OAuth
    EnableConfig --> Config
    OAuth --> MCP
    Config --> MCP
    MCP --> Plugins
    MCP --> Skills
    MCP --> GrowthBook
    
    %% REPL Launch
    GrowthBook --> Launch
    Launch --> State
    State --> UI

    %% Styling
    style CLI fill:#e3f2fd,stroke:#1976d2
    style Main fill:#e8f5e9,stroke:#388e3c
    style Init fill:#fff3e0,stroke:#f57c00
    style REPL fill:#fce4ec,stroke:#c2185b
```

## Key Files

| Component | File | Description |
|-----------|------|-------------|
| CLI Entry | `src/entrypoints/cli.tsx` | Fast path handler, zero-import optimizations |
| Main Entry | `src/main.tsx` | Parallel prefetch, heavy imports |
| Initialization | `src/entrypoints/init.ts` | Full app bootstrap |
| REPL Launch | `src/replLauncher.ts` | Screen initialization |
| State Store | `src/state/AppStateStore.ts` | Application state |

## Fast Paths (Zero Import)

These flags bypass full module loading:

- `--version` / `-v` / `-V` - Version output
- `--dump-system-prompt` - System prompt extraction
- `--claude-in-chrome-mcp` - Chrome extension MCP
- `--computer-use-mcp` - Computer automation MCP
- `--daemon-worker=<kind>` - Background workers

## Performance Notes

| Optimization | Savings |
|--------------|---------|
| Parallel prefetch (MDM + Keychain) | ~135ms |
| API preconnection | ~100-200ms |
| Fast path (--version) | Full import time |

---

*See also: [ARCHITECTURE.md](../ARCHITECTURE.md)*