# Flake Agent

AI agent that identifies flaky tests by analyzing patterns across synthetic Vitest JSON runs.
Phase 1: identification only. No triage or remediation.

## Commands

```bash
npm install              # install all dependencies
npm run build            # tsc → dist/
npm run test:run         # vitest run (single pass, non-interactive)
npm run lint             # eslint src/
npm run format           # prettier --write src/
```

Use `npm run test:run` not `npm test` — the latter starts vitest in watch mode which hangs in CI and agent contexts.

## Tech Stack

- TypeScript (strict, ES2022, ESNext modules, bundler resolution)
- Vitest for testing
- AI SDK (`ai` package) for the agent loop (ToolLoopAgent pattern)
- Zod for schema validation
- Node.js runtime, ES modules (`"type": "module"` in package.json)

## Architecture

Three planned packages (currently flat, workspace migration pending):

| Package | Responsibility | Key Interface |
|---------|---------------|---------------|
| simulator | Generates synthetic Vitest JSON test runs with configurable flake patterns | Produces `VitestJsonOutput` with ground truth |
| agent | Long-running process that analyzes runs and identifies flaky tests | Consumes runs, produces `FlakeReport` |
| eval | Measures agent accuracy (precision, recall, F1) against ground truth | Consumes reports + ground truth, produces metrics |

Data flows one direction: simulator → agent → eval.

ADRs in `docs/adr/` record key decisions. Read relevant ADRs before changing architecture.

## Vitest JSON Format

The simulator produces and the agent consumes the Vitest JSON reporter format. Key fields:

- `testResults[].assertionResults[].status` — "passed" | "failed" | "skipped"
- `testResults[].assertionResults[].fullName` — unique test identifier
- `testResults[].assertionResults[].duration` — milliseconds
- `testResults[].assertionResults[].failureMessages` — array of error strings

See ADR-0002 for the full schema and rationale.

## Flake Patterns to Support

The simulator must generate these failure modes (each has distinct detection signals):

- **Timing-dependent**: race conditions, timeouts — look for duration variance
- **Resource-dependent**: port conflicts, file locks — look for specific error messages
- **Order-dependent**: test pollution, shared state — look for pass/fail correlation with run order
- **Intermittent**: network, external services — look for random fail distribution

## Coding Conventions

- Imports: use `.js` extensions in TypeScript imports (required by ESNext module resolution)
- Schemas: define with Zod, infer TypeScript types with `z.infer<typeof Schema>`
- Errors: throw typed errors, never swallow silently
- Tests: colocate test files as `*.test.ts` next to source files
- No default exports except where required by a framework

## Gotchas

- `tsconfig.json` uses `"moduleResolution": "bundler"` — this means bare specifier imports resolve differently than Node16. Check import paths if you get resolution errors.
- The `ai` package (AI SDK) has a streaming-first API. Use `generateText()` for non-streaming agent loops.
- Vitest JSON output uses `startTime` as Unix epoch milliseconds, not ISO strings.
