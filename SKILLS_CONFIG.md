# OpenCode Skills Configuration

Skills customizadas para automatizar tarefas comuns de desenvolvimento.

## Overview

| Skill | Função | Triggers |
|-------|--------|----------|
| `run-tests` | Executa suites de teste | test, jest, vitest, pytest |
| `git-workflow` | Gerencia git operations | commit, push, branch, merge |
| `security-scan` | Escaneia vulnerabilidades | security, vulnerability, scan |

## File Structure

```
.opencode/
├── agents/                    # Agentes (12)
│   ├── code-writer.md         # Orchestrator (Standard)
│   ├── explore.md             # RAG Agent (Standard)
│   ├── router-nlp.md          # Router (Economy) ← NEW
│   ├── code-reviewer.md       # Reviewer (Premium)
│   ├── security-checker.md    # Security (Premium)
│   ├── bug-fixer.md           # Executor (Standard)
│   ├── tester.md              # Executor (Standard)
│   ├── python-writer.md       # Executor (Standard)
│   ├── code-documenter.md     # Support (Standard)
│   ├── dead-code-detector.md  # Support (Economy)
│   ├── documentation.md       # Support (Standard)
│   └── html-doc-generator.md  # Support (Standard)
├── skills/                    # Skills (3)
│   ├── run-tests.md
│   ├── git-workflow.md
│   └── security-scan.md
└── SKILLS_CONFIG.md           # Esta documentação
```

## Skills Details

### run-tests

**Description**: Executa suites de teste (unit, integration, e2e)

**Triggers**:
- "test", "run tests", "execute tests"
- "npm test", "jest", "vitest"
- "pytest", "ruby", "go test"

**Behavior**:
1. Detecta framework de teste
2. Executa comando apropriado
3. Reporta resultados (passed/failed)
4. Oferece fix em caso de falha

**Frameworks Suportados**:
- Node.js: Jest, Vitest
- Python: pytest
- Go: go test
- Ruby: rspec

### git-workflow

**Description**: Gerencia operações git (commits, branches, PRs)

**Triggers**:
- "commit", "push", "branch"
- "pull", "merge", "stash"
- "git"

**Behavior**:
1. Analisa mudanças (git status, git diff)
2. Gera commit message (conventional commits)
3. Confirma antes de executar
4. Executa operação

**Conventions**:
- Commit: `type(scope): description`
- Branch: `type/ticket-description`

### security-scan

**Description**: Escaneia vulnerabilidades de segurança

**Triggers**:
- "security", "vulnerability", "scan"
- "audit", "sast", "secrets"

**Behavior**:
1. Static analysis (SAST)
2. Dependency scan (npm audit, etc)
3. Secret detection
4. Gera relatório por severidade

**Severity Levels**:
- Critical: RCE, data breach
- High: Injection, auth bypass
- Medium: Information disclosure
- Low: Code smells

## Agent Permissions

| Agent | run-tests | git-workflow | security-scan |
|-------|-----------|--------------|---------------|
| code-writer (orchestrator) | allow | allow | allow |
| explore (RAG) | allow | deny | allow |
| router-nlp | deny | deny | deny |
| code-reviewer | allow | deny | allow |
| security-checker | allow | deny | allow |
| bug-fixer | allow | deny | allow |
| tester | allow | deny | deny |
| python-writer | allow | deny | allow |
| code-documenter | deny | deny | deny |
| dead-code-detector | deny | deny | deny |
| documentation | deny | deny | deny |
| html-doc-generator | deny | deny | deny |

## Auto-invocation

### Orchestrator Skills
The orchestrator (`code-writer`) detects triggers and invokes skills, then delegates to subagents:

```markdown
## code-writer.md (Orchestrator)
Quando detectar:
- "test" → @skill run-tests (then may call @tester)
- "commit" → @skill git-workflow
- "security" → @skill security-scan (then may call @security-checker)
```

### Subagent Skills
Executors and reviewers may invoke skills during their work:

```markdown
## code-reviewer.md
Quando detectar:
- "test" ou "coverage" → @skill run-tests
- "security" → @skill security-scan

## bug-fixer.md
Quando detectar:
- "test" ou "verify" → @skill run-tests
- "security bug" → @skill security-scan

## tester.md
Quando detectar:
- "test" → @skill run-tests (verify tests pass)

## python-writer.md
Quando detectar:
- "test", "pytest" → @skill run-tests
- "security" → @skill security-scan
```

## Usage

### Via @skill

```
@skill run-tests
@skill git-workflow commit
@skill security-scan --full
```

### Via trigger (auto)

```
"rode os testes" → auto-invoca run-tests
"commita isso" → auto-invoca git-workflow
"escaneia vulnerabilidades" → auto-invoca security-scan
```

## Output Format

### run-tests
```
=== Test Execution ===
Framework: Jest
Command: npm test

✓ 45 tests passed
✗ 2 tests failed
⏱ 12.5s
```

### git-workflow
```
=== Git Workflow ===
Status: 3 files changed
Suggested: feat(api): add login endpoint
Commit? [y/N]
```

### security-scan
```
=== Security Scan ===
Critical: 0
High: 2
Medium: 5
Low: 12

[HIGH] SQL Injection in src/db/query.ts:45
```

## Future Skills

Sugestões para implementar:

| Skill | Triggers | Função |
|-------|----------|--------|
| `lint-check` | "lint", "format", "prettier" | Roda linters |
| `typecheck` | "types", "typescript", "flow" | Verifica tipos |
| `build-project` | "build", "deploy", "compile" | Build e deploy |
| `dependency-check` | "deps", "outdated", "update" | Verifica deps |
| `generate-readme` | "readme", "docs" | Gera documentação |

## Configuration

Skills são configuradas via `opencode.json`:

```json
"skill": {
  "run-tests": "allow",
  "git-workflow": "allow",
  "security-scan": "allow"
}
```

Para desabilitar uma skill para um agente específico, use `deny`:

```json
"explore": {
  "skill": {
    "run-tests": "allow",
    "git-workflow": "deny",
    "security-scan": "allow"
  }
}
```