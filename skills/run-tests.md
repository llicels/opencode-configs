---
name: run-tests
description: Execute test suites (unit, integration, e2e)
triggers:
  - "test"
  - "run tests"
  - "execute tests"
  - "npm test"
  - "jest"
  - "vitest"
  - "pytest"
  - "ruby"
  - "go test"
---

# Run Tests Skill

Executes test suites for the project.

## Supported Test Frameworks

| Framework | Commands |
|-----------|----------|
| Node.js (Jest) | `npm test`, `npm run test:watch` |
| Node.js (Vitest) | `npm run test`, `vitest` |
| Python (pytest) | `pytest`, `python -m pytest` |
| Go | `go test ./...`, `go test -v` |
| Ruby | `rspec`, `rake test` |

## Behavior

1. **Detect test framework**: Check `package.json`, `pyproject.toml`, etc
2. **Execute appropriate command**: Based on detected framework
3. **Report results**: Summary of passed/failed tests
4. **Fix on failure**: Offer to fix failing tests

## Output Format

```
=== Test Execution ===

Framework: Jest
Command: npm test

Results:
✓ 45 tests passed
✗ 2 tests failed
⏱  12.5s

Failed tests:
  - should return 404 for invalid endpoint
  - should handle empty input

Fix failing tests? [y/N]
```

## Auto-detection Logic

```
if package.json contains "vitest" → use "vitest"
elif package.json contains "jest" → use "npm test"
elif pyproject.toml exists → use "pytest"
elif go.mod exists → use "go test ./..."
else → ask user for test command
```

## Options

| Option | Description |
|--------|-------------|
| `--watch` | Run tests in watch mode |
| `--coverage` | Generate coverage report |
| `--grep <pattern>` | Run tests matching pattern |
| `--fix` | Auto-fix failing tests |

## Notes

- Always run from project root
- Use `--passWithNoTests` if no tests exist
- Show coverage if available
- Parse error output to identify failing test locations