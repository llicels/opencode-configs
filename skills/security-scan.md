---
name: security-scan
description: Scan for security vulnerabilities in code
triggers:
  - "security"
  - "vulnerability"
  - "scan"
  - "audit"
  - "sast"
  - " secrets"
---

# Security Scan Skill

Scans code for security vulnerabilities and issues.

## Scan Types

### 1. Static Analysis (SAST)

Scans source code for patterns indicating security issues:

| Category | Patterns |
|----------|----------|
| Injection | SQL injection, XSS, command injection |
| Authentication | Weak passwords, hardcoded creds |
| Cryptography | Weak algorithms, exposed keys |
| Data Exposure | Sensitive data in logs, hardcoded secrets |

### 2. Dependency Scan

Checks for vulnerable dependencies:

```bash
# npm
npm audit
npm audit fix

# Python
pip-audit
safety check

# Go
go vet ./...
```

### 3. Secret Detection

Scans for exposed secrets:
- API keys
- Passwords
- Private keys
- Tokens

Tools: `git-secrets`, `trufflehog`, `detect-secrets`

## Behavior

1. **Run static analysis**: Parse code for vulnerability patterns
2. **Check dependencies**: Run `npm audit` or equivalent
3. **Scan for secrets**: Check for hardcoded credentials
4. **Generate report**: List findings by severity
5. **Suggest fixes**: Provide remediation guidance

## Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| Critical | RCE, data breach | Fix immediately |
| High | Injection, auth bypass | Fix within 24h |
| Medium | Information disclosure | Fix within week |
| Low | Code smells | Fix in next sprint |

## Output Format

```
=== Security Scan Results ===

Scan: Static Analysis + Dependencies
Files: 245 analyzed
Duration: 8.2s

Critical: 0
High: 2
Medium: 5
Low: 12

=== Findings ===

[HIGH] SQL Injection in src/db/query.ts:45
  - User input directly concatenated to query
  - Fix: Use parameterized queries

[MEDIUM] Hardcoded API Key in config/prod.js:12
  - API key exposed in source code
  - Fix: Use environment variables

[LOW] Console.log in src/utils/logger.ts
  - Sensitive data may be logged
  - Fix: Use proper logging library

=== Recommendations ===

1. Run npm audit fix
2. Implement parameterized queries
3. Move secrets to .env
```

## Tools Used

| Language | Tools |
|----------|-------|
| JavaScript/TypeScript | ESLint security plugin, npm audit |
| Python | bandit, safety |
| Go | go vet, gosec |
| General | truffleHog, detect-secrets |

## Options

| Option | Description |
|--------|-------------|
| `--full` | Full scan including secrets |
| `--fast` | Quick scan, basic patterns |
| `--fix` | Auto-fix where possible |
| `--report` | Generate HTML report |

## Notes

- Scan before commits (pre-commit hook suggestion)
- Prioritize fixing Critical/High issues
- Check for OWASP Top 10 vulnerabilities
- Warn about deprecated crypto functions