# Architecture Diagrams

This directory contains detailed architecture diagrams for each major component of Claude Code.

## Index

| Diagram | Description |
|---------|-------------|
| [startup.md](startup.md) | CLI entry, fast paths, parallel prefetch, initialization |
| [query-engine.md](query-engine.md) | Query loop, API calls, stream processing, tool execution |
| [security.md](security.md) | Input sanitization, permission checking, security pipeline |
| [tools.md](tools.md) | Tool system, categories, execution flow, interface |
| [commands.md](commands.md) | Command system, slash commands, command types |
| [services.md](services.md) | Service layer, MCP, plugins, analytics, compact |
| [state.md](state.md) | State management, AppStateStore, state structure |

## Usage

These diagrams are referenced from the main [ARCHITECTURE.md](../ARCHITECTURE.md). Each diagram provides:

- Detailed flow diagrams (Mermaid)
- Key files and components
- Performance notes where applicable

## Structure

```
docs/architecture/
├── README.md           # This file
├── startup.md          # Startup flow
├── query-engine.md     # Query engine
├── security.md         # Security pipeline
├── tools.md            # Tool system
├── commands.md         # Command system
├── services.md         # Service layer
└── state.md            # State management
```

## Related Docs

- [ARCHITECTURE.md](../ARCHITECTURE.md) - Main architecture document
- [prompt-harden.md](../prompt-harden.md) - Detailed security hardening doc