# Prompt Hardening & Input Sanitization Architecture

## Pipeline Diagram

```mermaid
flowchart TD
    A([User / Bridge Input]) --> B[processUserInput]

    B --> C{Bridge Origin?}
    C -- Yes --> D[isBridgeSafeCommand check\nskipSlashCommands enforced]
    C -- No --> E[Slash command parse]
    D --> F
    E --> F[Unicode Sanitization\nsanitization.ts\nNFKC + Cf/Co/Cn filter]

    F --> G[UserPromptSubmit Hooks\nhooks.ts\nTrust gate first]
    G --> H{Hook decision}
    H -- block --> I([Blocked — system message shown\noriginal input erased])
    H -- additionalContext\nmax 10 000 chars --> J[Augmented message]
    H -- pass --> J

    J --> K[LLM API call\nAnthropic SDK]

    K --> L[Tool use requested]
    L --> M[Permission Check\npermissions.ts]

    M --> N{Rule match?}
    N -- deny rule --> O([Blocked])
    N -- allow rule --> P([Execute tool])
    N -- no match --> Q{Dangerous pattern?\ndangerousPatterns.ts}
    Q -- match --> R{Permission mode}
    R -- default/ask --> S([Prompt user])
    R -- auto / YOLO --> T[Classifier LLM\nyoloClassifier.ts]
    T -- safe --> P
    T -- unsafe --> S
    Q -- no match --> P

    P --> U{MCP tool?}
    U -- Yes --> V[MCP output truncation\nmcpValidation.ts\n25 000 token cap]
    U -- No --> W[Tool result]
    V --> W

    W --> X[PreToolUse / PostToolUse Hooks\nTrust gate + timeout]
    X --> K
```

## Key Components

| Layer | File | Mechanism |
|---|---|---|
| Input Unicode hardening | `src/utils/sanitization.ts` | NFKC normalization + Unicode category filtering (Cf/Co/Cn) + explicit zero-width/RTL ranges |
| Permission gating | `src/utils/permissions/permissions.ts` | 3-tier: rule whitelist/blacklist → dangerous pattern detection → YOLO classifier (secondary LLM) |
| Dangerous pattern matching | `src/utils/permissions/dangerousPatterns.ts` | Cross-platform blocklist: `eval`, `exec`, `sudo`, `ssh`, `kubectl`, etc. |
| Hook execution gate | `src/utils/hooks.ts` | Trust dialog required before any hook runs; `shouldSkipHookDueToTrust()` |
| Hook output limit | `src/utils/processUserInput/processUserInput.ts` | Hook output capped at 10,000 chars before injecting into messages |
| MCP output truncation | `src/utils/mcpValidation.ts` | 25,000 token limit; image compression fallback |
| Permission rule validation | `src/utils/settings/validation.ts` | Escape-aware bracket parsing, Zod schema, per-rule filtering |
| User prompt hooks | `src/utils/processUserInput/processUserInput.ts` | `executeUserPromptSubmitHooks()` can block/augment/erase input pre-LLM |

## Notable Gaps

1. **Hook settings trust** — hooks are arbitrary shell commands from `.claude/settings.json`. If that file is tampered with, arbitrary code runs and permissions can be dynamically upgraded via `PermissionRequest` hooks.
2. **Hook output unsanitized** — hook `additionalContext` is injected into the message stream without Unicode sanitization, making it a potential prompt injection vector.
3. **Web API endpoints** (`web/app/api/files/read/route.ts`) — no authentication layer; any caller can request file reads if the server is reachable.
4. **Sanitization wiring** — `sanitization.ts` is comprehensive but worth verifying it is called on the main user prompt path and not only on MCP/tool results.
