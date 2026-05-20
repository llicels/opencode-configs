# OpenCode Agents Configuration

Configuration for OpenCode AI coding assistant with a tiered agent architecture for cost control and quality.

## Architecture Overview

This configuration implements a **3-tier model architecture** (Economy / Standard / Premium) with clear separation of concerns:

```
User Request
    │
    ▼
┌─────────────────┐
│  Orchestrator   │  ← code-writer (Standard: Qwen3.5 Plus)
│  (Primary)      │     Understands intent, routes to subagents
└────────┬────────┘
         │
    ┌────┴────┐
    ▼         ▼
┌────────┐  ┌─────────────┐
│ Router │  │  RAG Agent  │  ← explore (Standard: MiMo-V2.5)
│(Economy│  │  (Standard) │     Searches codebase, returns context
└────┬───┘  └─────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│              Executors (Standard)             │
│  ┌──────────┐ ┌────────┐ ┌──────────────┐  │
│  │bug-fixer │ │ tester │ │ python-writer │  │
│  │Kimi K2.6 │ │DeepSeek│ │  MiMo-V2.5    │  │
│  └──────────┘ │V4 Flash│ └──────────────┘  │
│               └────────┘                   │
└─────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│           Reviewers (Premium)                 │
│  ┌────────────────┐ ┌─────────────────────┐  │
│  │  code-reviewer │ │   security-checker  │  │
│  │  Qwen3.6 Plus  │ │   DeepSeek V4 Pro   │  │
│  └────────────────┘ └─────────────────────┘  │
└─────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│         Support Agents (Economy/Std)        │
│  ┌──────────────┐ ┌───────────────────────┐  │
│  │dead-code-detect│ │  code-documenter   │  │
│  │MiniMax M2.7   │ │   Kimi K2.6        │  │
│  └──────────────┘ └───────────────────────┘  │
│  ┌──────────────┐ ┌───────────────────────┐  │
│  │documentation │ │  html-doc-generator  │  │
│  │MiMo-V2.5     │ │   MiMo-V2.5        │  │
│  └──────────────┘ └───────────────────────┘  │
└─────────────────────────────────────────────┘
```

## Tier Classification

### Economy (cheap, fast, auxiliary tasks)
| Agent | Model | Use Case |
|-------|-------|----------|
| `router-nlp` | MiniMax M2.5 | Intent classification, routing decisions |
| `dead-code-detector` | MiniMax M2.7 | Dead code analysis (read-only) |

### Standard (balanced cost/quality — default for most flows)
| Agent | Model | Use Case |
|-------|-------|----------|
| `code-writer` (orchestrator) | Qwen3.5 Plus | Main orchestrator, routing decisions |
| `explore` (RAG) | MiMo-V2.5 | Codebase search and synthesis |
| `bug-fixer` | Kimi K2.6 | Bug diagnosis and fixing |
| `tester` | DeepSeek V4 Flash | Test writing and maintenance |
| `python-writer` | MiMo-V2.5 | Python code implementation |
| `code-documenter` | Kimi K2.6 | Inline documentation after changes |
| `documentation` | MiMo-V2.5 | Project documentation |
| `html-doc-generator` | MiMo-V2.5 | HTML/Markdown hybrid docs |

### Premium (maximum quality, use sparingly)
| Agent | Model | Use Case |
|-------|-------|----------|
| `code-reviewer` | Qwen3.6 Plus | Critical code review (high priority only) |
| `security-checker` | DeepSeek V4 Pro | Security scanning after every change |

## File Structure

```
.
├── opencode.json                    # Main configuration
└── .opencode/
    ├── agents/
    │   ├── code-writer.md           # Orchestrator (Standard)
    │   ├── explore.md               # RAG Agent (Standard)
    │   ├── router-nlp.md            # Router (Economy) ← NEW
    │   ├── code-reviewer.md         # Reviewer (Premium)
    │   ├── security-checker.md      # Security (Premium)
    │   ├── bug-fixer.md             # Executor (Standard)
    │   ├── tester.md                # Executor (Standard)
    │   ├── python-writer.md         # Executor (Standard)
    │   ├── code-documenter.md       # Support (Standard)
    │   ├── dead-code-detector.md    # Support (Economy)
    │   ├── documentation.md         # Support (Standard)
    │   └── html-doc-generator.md    # Support (Standard)
    ├── skills/
    │   ├── run-tests.md
    │   ├── git-workflow.md
    │   └── security-scan.md
    ├── OPENCODE_AGENTS_CONFIG.md    # This file
    └── SKILLS_CONFIG.md
```

## Agents Summary

| Agent | Type | Tier | Model | Description |
|-------|------|------|-------|-------------|
| `code-writer` | primary | Standard | Qwen3.5 Plus | **Orchestrator** — understands intent, routes to subagents, assembles responses. Never writes code directly. |
| `explore` | primary | Standard | MiMo-V2.5 | **RAG Agent** — searches codebase, synthesizes answers using only provided context with citations |
| `router-nlp` | subagent | Economy | MiniMax M2.5 | **Intent Classifier** — outputs strict JSON with intent, priority, and suggested agent |
| `code-reviewer` | subagent | Premium | Qwen3.6 Plus | **Critical Reviewer** — reviews high-priority code. Concise, actionable feedback |
| `security-checker` | subagent | Premium | DeepSeek V4 Pro | **Security Reviewer** — automatically invoked after EVERY code modification |
| `bug-fixer` | subagent | Standard | Kimi K2.6 | **Executor** — fixes bugs based on orchestrator instructions |
| `tester` | subagent | Standard | DeepSeek V4 Flash | **Executor** — writes tests based on orchestrator instructions |
| `python-writer` | subagent | Standard | MiMo-V2.5 | **Executor** — writes Python code based on orchestrator instructions |
| `code-documenter` | subagent | Standard | Kimi K2.6 | **Support** — automatically invoked after EVERY code modification to add docstrings |
| `dead-code-detector` | subagent | Economy | MiniMax M2.7 | **Support** — finds dead/unused code (read-only) |
| `documentation` | subagent | Standard | MiMo-V2.5 | **Support** — writes project documentation |
| `html-doc-generator` | subagent | Standard | MiMo-V2.5 | **Support** — generates HTML/Markdown hybrid docs |

## Workflow

### Standard Flow

1. **User sends request** → `code-writer` (orchestrator)
2. **Orchestrator calls** `@router-nlp` → receives JSON with intent + suggested agent
3. **Orchestrator invokes** the appropriate subagent(s):
   - Code tasks → `@python-writer`, `@bug-fixer`, `@tester`
   - Review tasks → `@code-reviewer` (if priority: alta)
   - Search tasks → `@explore`
   - Docs tasks → `@documentation`, `@code-documenter`, `@html-doc-generator`
   - Cleanup tasks → `@dead-code-detector`
4. **After ANY code modification**, orchestrator automatically invokes:
   - `@security-checker` (mandatory security scan)
   - `@code-documenter` (mandatory documentation)
5. **Orchestrator assembles** final response from subagent results

### Emergency / High Priority Flow

If `router-nlp` returns `priority: alta`:
1. Orchestrator may invoke `@code-reviewer` (Premium) for critical review
2. Security scan is especially thorough
3. Fallback to Premium models if Standard fails

## Permissions Reference

### Permission Values

| Value | Behavior |
|-------|----------|
| `allow` | Tool available without confirmation |
| `ask` | User confirmation required before use |
| `deny` | Tool unavailable |

### Agent Permissions Matrix

| Agent | edit | write | bash | read | grep | glob | patch | webfetch | skills |
|-------|------|-------|------|------|------|------|-------|----------|--------|
| code-writer (orchestrator) | allow | allow | ask | allow | allow | allow | allow | deny | run-tests, git-workflow, security-scan |
| explore (RAG) | deny | deny | deny | allow | allow | allow | deny | deny | run-tests, security-scan |
| router-nlp | deny | deny | deny | allow | allow | allow | deny | deny | deny |
| code-reviewer | deny | deny | deny | allow | allow | allow | deny | deny | run-tests, security-scan |
| security-checker | deny | deny | deny | allow | allow | allow | deny | deny | security-scan, run-tests |
| bug-fixer | ask | ask | deny | allow | allow | allow | allow | deny | run-tests, security-scan |
| tester | allow | allow | deny | allow | allow | allow | allow | deny | run-tests |
| python-writer | allow | allow | ask | allow | allow | allow | allow | deny | run-tests, security-scan |
| code-documenter | allow | allow | deny | allow | allow | allow | deny | deny | deny |
| dead-code-detector | deny | deny | deny | allow | allow | allow | deny | deny | deny |
| documentation | allow | allow | deny | allow | allow | allow | deny | deny | deny |
| html-doc-generator | allow | allow | deny | allow | allow | allow | deny | deny | deny |

## Cost Control Strategy

### Daily Budget Allocation (suggested)

| Tier | % of Budget | Agents |
|------|-------------|--------|
| Economy | 40-60% | router-nlp, dead-code-detector |
| Standard | 30-40% | code-writer, explore, bug-fixer, tester, python-writer, code-documenter, documentation, html-doc-generator |
| Premium | 5-15% | code-reviewer, security-checker |

### Per-Agent Limits

| Agent | Max Tokens/Call | Daily Budget % |
|-------|-----------------|----------------|
| code-writer | 1.5k-2k | 30-40% |
| router-nlp | 512-700 | 5-10% |
| explore | 1.5k-2k | 20-30% |
| bug-fixer | up to 1k | 10-20% |
| tester | up to 1.5k | 10-20% |
| python-writer | up to 1.5k | 10-20% |
| code-reviewer | 1.5k-2k | 5-10% |
| security-checker | 1.5k-2k | 5-10% |
| code-documenter | up to 1.5k | 5-10% |
| dead-code-detector | up to 1k | 2-5% |
| documentation | up to 1.5k | 5-10% |
| html-doc-generator | up to 1.5k | 5-10% |

### Fallback & Degradation

If approaching limits:
1. **Premium → Standard**: Use Standard model with shorter responses
2. **Standard → Economy**: Use Economy model with simplified output
3. **Last resort**: Return "Daily capacity reached, try again later"

For internal/auxiliary tasks (router, dead-code detection): **always use Economy**.

## Invoking Agents

### Via Orchestrator (Recommended)
The orchestrator (`code-writer`) automatically routes to the correct subagent. Just describe what you need:

```
"Fix the login bug" → router → bug-fixer → security-checker + code-documenter
"Write tests for the API" → router → tester → security-checker + code-documenter
"Review this Python code" → router → code-reviewer
```

### Manual Invocation
In your messages, use @mention to invoke specific agents:

```
@router-nlp "Classify this request"
@explore "Where is the auth logic?"
@python-writer "Implement a Django model"
@bug-fixer "Fix the login issue"
@tester "Write tests for the API"
@code-reviewer "Review the auth module"
@security-checker "Scan the new endpoints"
@dead-code-detector "Find unused code in utils"
@code-documenter "Add docstrings to models.py"
@documentation "Update the README"
@html-doc-generator "Generate API docs"
```

## Auto-Invocation Rules

### Always Auto-Invoked After Code Changes
- `@security-checker` — scans all modified files for vulnerabilities
- `@code-documenter` — adds docstrings to new/modified functions

### Skill Auto-Detection
Agents detect triggers and invoke skills:
- "test", "run tests", "jest", "vitest" → `@skill run-tests`
- "commit", "push", "branch", "git" → `@skill git-workflow`
- "security", "vulnerability", "scan", "audit" → `@skill security-scan`

## Configuration

### Default Agent
```json
"default_agent": "code-writer"
```

### Model Configuration
Each agent has a specific paid model assigned based on its tier. See `opencode.json` for the full configuration.

| Agent | Model | Tier |
|-------|-------|------|
| code-writer | `opencode/qwen3.5-plus` | Standard |
| explore | `opencode/mimo-v2.5` | Standard |
| router-nlp | `opencode/minimax-m2.5` | Economy |
| code-reviewer | `opencode/qwen3.6-plus` | Premium |
| security-checker | `opencode/deepseek-v4-pro` | Premium |
| bug-fixer | `opencode/kimi-k2.6` | Standard |
| tester | `opencode/deepseek-v4-flash` | Standard |
| python-writer | `opencode/mimo-v2.5` | Standard |
| code-documenter | `opencode/kimi-k2.6` | Standard |
| dead-code-detector | `opencode/minimax-m2.7` | Economy |
| documentation | `opencode/mimo-v2.5` | Standard |
| html-doc-generator | `opencode/mimo-v2.5` | Standard |

## Notes

- **Orchestrator never writes code directly** — always delegates to executors
- **Router-nlp is always called first** for non-trivial requests
- **Security-checker and code-documenter are auto-invoked after EVERY code modification**
- Subagents send their findings back to the orchestrator
- The orchestrator coordinates all work and makes final decisions
- All agents can read code but have different write permissions
- `webfetch` is disabled for all agents (security)
- `bash` is restricted to orchestrator and python-writer only (ask mode)
- `security-checker` and `code-documenter` are mandatory post-change steps
- `dead-code-detector` and `security-checker` are read-only (never modify code)
- Use Economy tier for internal/auxiliary tasks to control costs
- Reserve Premium tier for high-risk flows where errors are expensive
