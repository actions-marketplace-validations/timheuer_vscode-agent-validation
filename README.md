# Validate VS Code Agent

[![CI](https://github.com/timheuer/vscode-agent-validation/actions/workflows/ci.yml/badge.svg)](https://github.com/timheuer/vscode-agent-validation/actions/workflows/ci.yml)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

A GitHub Action to validate `.agent.md` files against the [VS Code custom agent specification](https://code.visualstudio.com/docs/copilot/customization/custom-agents).

## Why Use This Action?

- **Spec compliance** - Ensure your custom agents follow the official VS Code specification
- **Catch errors early** - Validate agents in CI before they reach production
- **Cross-platform** - Works with VS Code agents for both local and GitHub Copilot targets
- **Configurable** - Ignore specific rules, validate file references, or run in strict mode
- **CI-friendly outputs** - JSON arrays for errors and warnings for programmatic use

## Usage

### Basic Usage

```yaml
- name: Validate VS Code Agents
  uses: timheuer/vscode-agent-validation@v1
  with:
    path: .
```

See the [examples](.github/examples) folder for workflow templates:

- [Simple example](.github/examples/simple.yml) - Basic validation of all agent files
- [Advanced example](.github/examples/advanced.yml) - Validate only changed files with strict mode

### Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to `.agent.md` file or directory containing agent files | No | `.` |
| `fail-on-warning` | Treat warnings as errors | No | `false` |
| `ignore-rules` | Comma-separated list of rule IDs to ignore | No | `""` |
| `validate-references` | Check that file references in agent body actually exist | No | `false` |

### Outputs

| Output | Description |
|--------|-------------|
| `valid` | Whether the agent file(s) are valid (`true`/`false`) |
| `errors` | JSON array of validation errors |
| `warnings` | JSON array of validation warnings |
| `files-validated` | Number of agent files validated |

### Example: Using Outputs

```yaml
- name: Validate Agents
  id: validate
  uses: timheuer/vscode-agent-validation@v1
  with:
    path: .github/agents

- name: Check results
  if: always()
  run: |
    echo "Valid: ${{ steps.validate.outputs.valid }}"
    echo "Errors: ${{ steps.validate.outputs.errors }}"
    echo "Warnings: ${{ steps.validate.outputs.warnings }}"
```

### Example: Ignore Specific Rules

```yaml
- name: Validate Agents
  uses: timheuer/vscode-agent-validation@v1
  with:
    path: .github/agents
    ignore-rules: description-quality,body-empty
```

### Example: Strict Mode (Fail on Warnings)

```yaml
- name: Validate Agents (strict)
  uses: timheuer/vscode-agent-validation@v1
  with:
    path: .github/agents
    fail-on-warning: "true"
```

### Example: Validate File References

```yaml
- name: Validate Agents with References
  uses: timheuer/vscode-agent-validation@v1
  with:
    path: .github/agents
    validate-references: "true"
```

## What Gets Validated

This action validates your `.agent.md` files against the [VS Code custom agent specification](https://code.visualstudio.com/docs/copilot/customization/custom-agents#_custom-agent-file-structure).

### Structure

- YAML frontmatter must be present (between `---` markers)
- Frontmatter must be valid YAML

### Frontmatter Fields

| Field | Validation | Severity |
|-------|-----------|----------|
| `description` | Should be a non-empty string | Warning |
| `name` | Must be a string if provided | Error |
| `argument-hint` | Must be a string if provided | Error |
| `tools` | Must be an array of strings | Error |
| `agents` | Must be array of strings, `*`, or `[]` | Error |
| `model` | Must be string or array of strings | Error |
| `user-invokable` | Must be boolean if provided | Error |
| `disable-model-invocation` | Must be boolean if provided | Error |
| `target` | Must be `vscode` or `github-copilot` | Error |
| `mcp-servers` | Must be an array if provided | Error |
| `handoffs` | Must be array with `label` and `agent` per item | Error |
| `infer` | Deprecated, use alternatives | Warning |

### Handoffs

Each handoff object is validated for:

- Required `label` field (string)
- Required `agent` field (string)
- Optional `prompt` field (string)
- Optional `send` field (boolean)
- Optional `model` field (string)

### Warnings

The action also produces warnings for:

- Very short descriptions (< 50 characters)
- Empty body content
- Body exceeding 1000 lines
- Unknown fields in frontmatter

## Validation Rules

| Rule ID | Severity | Description |
|---------|----------|-------------|
| `frontmatter-required` | Error | YAML frontmatter must exist |
| `frontmatter-valid` | Error | YAML must parse correctly |
| `file-extension` | Error | File must have `.agent.md` extension |
| `description-format` | Warning | Description should be non-empty string |
| `description-quality` | Warning | Description should be 50+ characters |
| `name-format` | Error | Name must be string if provided |
| `argument-hint-format` | Error | Argument-hint must be string if provided |
| `tools-format` | Error | Tools must be array of strings |
| `agents-format` | Error | Agents must be array, `*`, or `[]` |
| `model-format` | Error | Model must be string or string array |
| `user-invokable-format` | Error | Must be boolean |
| `disable-model-invocation-format` | Error | Must be boolean |
| `infer-deprecated` | Warning | Use newer fields instead |
| `target-valid` | Error | Must be `vscode` or `github-copilot` |
| `mcp-servers-format` | Error | Must be array of configs |
| `handoffs-format` | Error | Must be array of handoff objects |
| `handoff-label-required` | Error | Each handoff needs a label |
| `handoff-agent-required` | Error | Each handoff needs target agent |
| `handoff-send-format` | Error | Send must be boolean |
| `handoff-model-format` | Error | Model must be string |
| `unknown-field` | Warning | Unknown frontmatter field detected |
| `body-empty` | Warning | Body content is empty |
| `body-too-long` | Warning | Body exceeds 1000 lines |
| `reference-not-found` | Error | Referenced file doesn't exist |

## Valid .agent.md Example

```markdown
---
description: A planning agent that researches and creates implementation plans
name: planner
argument-hint: Describe the feature or problem to plan
tools:
  - search
  - fetch
  - read_file
agents:
  - "*"
model: claude-sonnet-4
user-invokable: true
handoffs:
  - label: Start Implementation
    agent: implementation
    prompt: Implement the plan above.
    send: false
---

# Planning Agent

You are a planning specialist focused on research and planning.

## Guidelines

1. Research the codebase before planning
2. Break down tasks into clear steps
3. Consider edge cases and dependencies
```

## File Discovery

The action searches for agent files in the following locations:

1. `.github/agents/` directory (standard location)
2. Root directory of the specified path
3. `.github/chatmodes/` for legacy `.chatmode.md` files

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## Acknowledgments

This project was inspired by [validate-skill](https://github.com/Flash-Brew-Digital/validate-skill) by [Flash Brew Digital](https://flashbrew.digital/), which validates SKILL.md files against the Agent Skills specification.

## License

[MIT License](LICENSE)

## Author

[Tim Heuer](https://github.com/timheuer)
