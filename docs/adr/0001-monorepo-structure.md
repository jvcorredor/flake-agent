# ADR-0001: Monorepo Structure with npm Workspaces

## Status

Accepted

## Context

The flake-agent project has three distinct components:

1. **Test Run Simulator** - Generates synthetic test data
2. **Flake Detection Agent** - Analyzes test runs to identify flaky tests
3. **Evaluation Framework** - Measures agent performance against ground truth

Each component has clear responsibilities and interfaces:
- Simulator produces test runs (Vitest JSON format)
- Agent consumes test runs, produces flake reports
- Eval consumes flake reports and ground truth, produces metrics

We need a project structure that:
- Keeps components isolated for independent development
- Allows clear contracts between packages
- Supports different dependency profiles (simulator needs no AI SDK)
- Enables easy testing of each component in isolation

## Decision

Use a monorepo structure with **npm workspaces** containing three packages:

- `@flake-agent/simulator` - Test run generator
- `@flake-agent/agent` - Flake detection agent (core product)
- `@flake-agent/eval` - Evaluation framework

Each package has its own `package.json` with package-specific dependencies. The root `package.json` defines workspaces and shared dev dependencies.

## Alternatives Considered

### Single Package
- **Pros:** Simpler setup, no workspace configuration
- **Cons:** Tightly coupled components, harder to test in isolation, single dependency profile

### pnpm Workspaces
- **Pros:** Better dependency isolation, faster installs, disk-efficient
- **Cons:** Additional tool to learn, less familiar to most JS developers

### Turborepo
- **Pros:** Task orchestration, caching, parallel execution
- **Cons:** Overkill for 3 packages, adds complexity we don't need yet

## Consequences

### Positive
- Clear separation of concerns between components
- Independent testing and development per package
- Can version/deploy packages independently if needed
- Familiar tooling (npm workspaces is built into npm 7+)
- Single lock file for consistent dependency versions

### Negative
- Slightly more complex project setup
- Hoisted dependencies may cause version conflicts (mitigated by keeping dependencies compatible)
- Need to run commands with `-w` flag to target specific packages

### Neutral
- All packages share root `node_modules` by default
- Need to be intentional about which dependencies are shared vs. package-specific
