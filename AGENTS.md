# VS Code Agent Validator - Agent Instructions

This repository contains a GitHub Action that validates `.agent.md` files against the VS Code custom agent specification.

## Project Structure

- `src/` - TypeScript source code
  - `index.ts` - Main entry point and validator logic
  - `types.ts` - TypeScript type definitions
  - `rules.json` - Validation rule definitions
- `test/` - Test files and fixtures
  - `fixtures/valid/` - Valid agent files for testing
  - `fixtures/invalid/` - Invalid agent files for testing
  - `test.ts` - Test runner
- `dist/` - Compiled output (built with ncc)
- `.github/` - GitHub workflows and examples

## Development Workflow

1. Install dependencies: `npm install`
2. Make changes to `src/` files
3. Type check: `npm run type-check`
4. Build: `npm run build`
5. Test: `npm test`

## Key Concepts

### Validation Rules

Rules are defined in `src/rules.json` with:

- `id` - Unique identifier used for ignoring rules
- `severity` - `error` or `warning`
- `message` - Template with `{placeholder}` substitutions

### Agent File Structure

VS Code agents have:

- YAML frontmatter between `---` markers
- Required: `description` field
- Optional: name, tools, agents, model, handoffs, target, mcp-servers, etc.
- Markdown body with agent instructions

### Adding New Rules

1. Add rule definition to `src/rules.json`
2. Add `RuleId` union type member in `src/types.ts`
3. Implement validation in `src/index.ts`
4. Add test fixtures in `test/fixtures/`

## Testing Locally

```bash
npm run build
npm test
```

Set `INPUT_PATH` environment variable to test different paths.
