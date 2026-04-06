# State Management Architecture

> **Reference**: Main diagram in [ARCHITECTURE.md](../ARCHITECTURE.md)

## Overview

The State Management system provides a centralized, immutable store for all application state.

## State Architecture

```mermaid
flowchart TB
    subgraph Store["AppStateStore"]
        Get["getState()"]
        Set["setState(updater)"]
        Subscribe["subscribe(listener)"]
    end

    subgraph State["Application State"]
        Settings["settings\nUser preferences"]
        Model["mainLoopModel\nSelected AI model"]
        MCP["mcp\nclients · tools · commands"]
        Plugins["plugins\nenabled · disabled"]
        Tasks["tasks\nBackground tasks"]
        Perms["toolPermissionContext\nPermission rules"]
        AgentReg["agentNameRegistry\nNamed agents"]
        History["fileHistory\nEdit tracking"]
        Todos["todos\nPer-agent todos"]
    end

    subgraph SideEffects["Side Effects"]
        Persist["Persist cost"]
        Notify["Notify IDE bridge"]
        Update["Update UI"]
    end

    Get --> State
    Set --> State
    Subscribe --> SideEffects
```

## State Structure

```typescript
type AppState = {
  // Core
  settings: SettingsJson
  mainLoopModel: ModelSetting
  
  // Integration
  mcp: {
    clients: Map<string, MCPClient>
    tools: Tool[]
    commands: Command[]
    resources: McpResource[]
    pluginReconnectKey: string
  }
  
  // Extensions
  plugins: {
    enabled: string[]
    disabled: string[]
    commands: Command[]
    errors: PluginError[]
    installationStatus: InstallationStatus
    needsRefresh: boolean
  }
  
  // Tasks
  tasks: { [taskId: string]: TaskState }
  
  // Permissions
  toolPermissionContext: ToolPermissionContext
  
  // Coordination
  agentNameRegistry: Map<string, AgentId>
  
  // File State
  fileHistory: FileHistoryState
  
  // Todos
  todos: { [agentId: string]: TodoList }
  
  // ... more fields
}
```

## State Update Pattern

```mermaid
sequenceDiagram
    participant C as Component
    participant S as AppStateStore
    participant U as UI/Render

    C->>S: setState(prev => newState)
    S->>S: immutable update
    S->>S: notify listeners
    S->>U: trigger re-render
    U-->>C: display new state
```

## Key Patterns

| Pattern | Description |
|---------|-------------|
| **Immutable updates** | `setState(updater)` returns new state |
| **Pub-sub** | `subscribe(listener)` returns unsubscribe fn |
| **Side-effects** | `onChangeAppState()` handles persistence |

## Key Files

| Component | File | Description |
|-----------|------|-------------|
| Store | `src/state/store.ts` | Simple pub-sub store |
| AppState | `src/state/AppStateStore.ts` | Full app state (~570 lines) |
| Hook | `src/hooks/useAppState.ts` | React hook for state |

---

*See also: [ARCHITECTURE.md](../ARCHITECTURE.md)*