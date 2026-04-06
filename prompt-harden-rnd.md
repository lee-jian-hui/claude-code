# Prompt Hardening R&D: Claude Code vs Industry Advances

> **Reference**: Claude Code implementation in [docs/architecture/security.md](docs/architecture/security.md)

## Executive Summary

This document compares Claude Code's prompt hardening implementation against recent advances in the field (2024-2026). It identifies gaps, opportunities for improvement, and relevant research/tools.

## Table of Contents
1. [Current Claude Code Implementation](#current-claude-code-implementation)
2. [Recent Research & Advances](#recent-research--advances)
3. [Comparison Analysis](#comparison-analysis)
4. [Recommendations](#recommendations)
5. [References](#references)

---

## Current Claude Code Implementation

### Security Pipeline (As of 2026-04)

| Layer | Technique | Implementation |
|-------|-----------|----------------|
| **Unicode Sanitization** | NFKC normalization + Unicode category filtering (Cf/Co/Cn) + explicit zero-width/RTL removal | `src/utils/sanitization.ts` |
| **Bridge Validation** | `isBridgeSafeCommand()` check | `src/commands.ts` |
| **Dangerous Patterns** | Shell command blocklist (eval, exec, sudo, ssh, kubectl) | `src/utils/permissions/dangerousPatterns.ts` |
| **Hook Execution** | Trust gate + output limits (10k chars) | `src/utils/hooks.ts` |
| **MCP Validation** | Token capping (25k tokens) | `src/utils/mcpValidation.ts` |
| **YOLO Classifier** | Secondary LLM for auto mode | `src/utils/permissions/yoloClassifier.ts` |

### Permission Modes
- `default` - Prompt for destructive operations
- `plan` - Read-only
- `bypassPermissions` - All tools allowed
- `auto` - Follow allow/deny/ask rules

### Known Gaps
1. 🔴 **Hook output sanitization** - `additionalContext` not sanitized
2. 🟠 **Web API auth** - `web/app/api/*` routes lack authentication
3. 🟠 **Settings integrity** - No hash verification on `settings.json`

---

## Recent Research & Advances

### 1. Academic Papers (2024-2026)

| Paper | Year | Key Contribution |
|-------|------|------------------|
| **"Defeating Prompt Injections by Design"** (arXiv:2503.18813) | 2025 | Architectural approach using separate LLM instances for untrusted content |
| **"A Critical Evaluation of Defenses against Prompt Injection Attacks"** (arXiv:2505.18333) | 2025 | Systematic evaluation showing most defenses can be bypassed |
| **"SecInfer: Preventing Prompt Injection via Inference-time Scaling"** (arXiv:2509.24967) | 2025 | Inference-time detection using LLM's internal representations |
| **"PIShield: Detecting Prompt Injection Attacks via Intrinsic LLM Features"** (arXiv:2510.14005) | 2025 | Detection using LLM's internal attention patterns |
| **"Defending Against Prompt Injection With a Few DefensiveTokens"** (ACM AISec 2025) | 2025 | Embedding invisible "defensive tokens" to detect injections |
| **"Systematic Literature Review on LLM Defenses"** (arXiv:2601.22240) | 2026 | Comprehensive taxonomy of 180+ defense methods |

### 2. Industry Frameworks & Tools

| Tool/Framework | Organization | Focus |
|----------------|--------------|-------|
| **Guardrails AI** | Guardrails AI | Input/output validation framework |
| **NeMo Guardrails** | NVIDIA | Topic control, input/output validation |
| **LLM Guard** | Lakera | Prompt injection detection, content filtering |
| **AI Safety Toolkit** | CISPA | Multi-layer defense framework |
| **ClawGuard** | Open Source | Regex-based prompt injection scanner |
| **PromptSonar** | Security Vendor | Static analyzer for LLM prompt vulnerabilities |
| **navi-sanitize** | Project-Navi | Deterministic sanitization (homoglyphs, invisible chars) |

### 3. Major Defense Strategies (2025-2026)

| Strategy | Description | Effectiveness |
|----------|-------------|---------------|
| **Architectural Separation** | Use separate LLM instances for untrusted content | High |
| **Token-Level Detection** | ML classifiers for injection patterns | Medium-High |
| **Inference-Time Detection** | Use LLM's internal representations | High |
| **Defensive Tokens** | Invisible markers to detect manipulation | Medium |
| **Multi-Layer Defense** | Defense in depth (input → processing → output) | High |
| **Prompt Encryption** | Encrypt sensitive prompt sections | Medium |

### 4. OWASP Guidelines

**OWASP LLM Top 10 (2025) - LLM01: Prompt Injection**
- Direct prompt injection (from user)
- Indirect prompt injection (from external data)
- Chain-of-thought exploitation

[Reference: OWASP Gen AI Security Project](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)

---

## Comparison Analysis

### Where Claude Code is Strong ✅

| Category | Claude Code Approach | Industry Status |
|----------|---------------------|-----------------|
| **Unicode Sanitization** | NFKC + category filtering | Industry standard, similar to navi-sanitize |
| **Permission Modes** | 4-mode system (default/plan/bypass/auto) | More granular than most |
| **Hook Trust Gate** | Requires trust dialog before execution | Not commonly implemented |
| **MCP Token Capping** | 25k token limit on MCP output | Good practice, not universal |
| **Dangerous Pattern Blocklist** | Shell command patterns | Standard approach |

### Gaps & Opportunities ❌

| Gap | Severity | Industry Advance | Recommendation |
|-----|----------|------------------|----------------|
| **Hook output sanitization** | 🔴 High | New in 2025 - critical gap | Apply sanitization.ts to hook output |
| **No ML-based classifier** | 🔴 High | PIShield, SecInfer use LLM features | Consider internal representation detection |
| **No input/output guardrails framework** | 🟠 Medium | Guardrails AI, NeMo are standard | Evaluate integration |
| **Static blocklist only** | 🟠 Medium | ML classifiers outperform regex | Add pattern learning |
| **No architectural separation** | 🟠 Medium | "Defeating Prompt Injections by Design" | Consider sandboxed LLM for untrusted |
| **No defensive tokens** | 🟡 Low | New research (ACM 2025) | Monitor research |

### Missing Techniques

| Technique | Reference | Claude Code Status |
|-----------|-----------|-------------------|
| **Embedding-level detection** | PIShield (internal features) | Not implemented |
| **Inference-time scaling** | SecInfer | Not implemented |
| **Prompt structure markers** | "System Boundaries" paper | Not implemented |
| **Separate LLM for untrusted** | Debenedetti 2025 | Not implemented |
| **Output sanitization** | General best practice | Limited implementation |

---

## Recommendations

### Short Term (High Priority)

1. **Fix hook output sanitization gap**
   ```typescript
   // Apply sanitization to hook additionalContext
   function sanitizeHookOutput(output: string): string {
     return partiallySanitizeUnicode(output);
   }
   ```

2. **Add input guardrails integration**
   - Evaluate LLM Guard or Guardrails AI
   - Add structured input validation

3. **Enhance dangerous pattern detection**
   - Add ML-based classifier alongside blocklist
   - Consider fine-tuned model for pattern detection

### Medium Term

4. **Implement architectural separation**
   - Separate processing for untrusted external content
   - Consider sandboxed LLM instances

5. **Add output sanitization**
   - Sanitize tool outputs before display
   - Filter sensitive patterns in responses

6. **Add defensive tokens (experimental)**
   - Embed invisible markers in prompts
   - Detect if markers appear in model outputs

### Long Term

7. **Evaluate inference-time detection**
   - Monitor PIShield, SecInfer research
   - Consider internal representation detection

8. **Add observability**
   - Log security events
   - Track blocked attacks
   - Measure defense effectiveness

---

## References

### Academic Papers

| Citation | URL |
|----------|-----|
| Debenedetti, "Defeating Prompt Injections by Design" (2025) | https://arxiv.org/abs/2503.18813 |
| Jia & Shao, "Critical Evaluation of Defenses" (2025) | https://arxiv.org/abs/2505.18333 |
| Liu et al., "SecInfer" (2025) | https://arxiv.org/abs/2509.24967 |
| "PIShield" (2025) | https://arxiv.org/abs/2510.14005 |
| "Systematic Literature Review" (2026) | https://arxiv.org/abs/2601.22240 |

### Industry Resources

| Resource | URL |
|----------|-----|
| OWASP LLM01:2025 | https://genai.owasp.org/llmrisk/llm01-prompt-injection/ |
| Google Layered Defense (2025) | https://security.googleblog.com/2025/06/mitigating-prompt-injection-attacks.html |
| Microsoft Indirect Injection Defense | https://www.microsoft.com/en-us/msrc/blog/2025/07/how-microsoft-defends-against-indirect-prompt-injection-attacks |
| Cisco "Guardrails Aren't Enough" | https://blogs.cisco.com/ai/prompt-injection-is-the-new-sql-injection-and-guardrails-arent-enough |

### Tools & Libraries

| Tool | URL |
|------|-----|
| navi-sanitize | https://github.com/Project-Navi/navi-sanitize |
| Guardrails AI | https://www.guardrailsai.com/ |
| NeMo Guardrails | https://github.com/NVIDIA/NeMo-Guardrails |
| LLM Guard | https://github.com/kaist-ina/llmguard |
| ClawGuard (regex scanner) | https://dev.to/joergmichno/12-ways-attackers-bypass-prompt-injection-scanners |

### Additional Reading

| Article | URL |
|---------|-----|
| "Unicode-Based LLM Attacks" (2026) | https://thehgtech.com/guides/unicode-llm-attacks.html |
| "Building Production Input Sanitizer" | https://redteams.ai/topics/walkthroughs/defense/building-input-sanitizer |
| "12 Ways to Bypass Scanners" | https://dev.to/joergmichno/12-ways-attackers-bypass-prompt-injection-scanners |

---

## Conclusion

Claude Code's current implementation covers **baseline security measures** that align with industry best practices from 2023-2024. However, the rapidly evolving threat landscape (2025-2026) has introduced **more sophisticated attacks** that may bypass traditional defenses.

**Key findings:**
1. Unicode sanitization is solid (matches current standards)
2. Hook output gap is critical and should be fixed
3. ML-based detection is not implemented but widely recommended
4. Architectural changes (separate LLM instances) are the most effective but require significant refactoring

**Priority actions:**
1. Fix hook output sanitization immediately
2. Add ML-based detection layer
3. Evaluate guardrails framework integration

---

*Last updated: 2026-04-06*
*Related docs: [ARCHITECTURE.md](../ARCHITECTURE.md), [docs/architecture/security.md](docs/architecture/security.md), [prompt-harden.md](prompt-harden.md)*