# Flake Agent

## Problem

Flaky tests erode trust in CI, waste developer time, and slow down development cycles. Identifying which tests are flaky requires analyzing patterns across many test runs - a tedious, error-prone process that's well-suited for automation.

This project explores building a **long-running agent that identifies flaky tests** by analyzing CI test output patterns. This is Phase 1: identification only. Triage and remediation come later.

**Challenge:** To evaluate whether the agent works, we need realistic test data with known flaky behavior. Since no such dataset exists, we must create a synthetic test run simulator that generates test output with configurable flake patterns.

## Approach

```
┌─────────────────────────┐
│  Test Run Simulator     │  Generates synthetic Vitest JSON output
│  - Configurable flakes   │  with known flaky patterns
│  - Ground truth tracking │
└───────────┬─────────────┘
            │ test runs
            ▼
┌─────────────────────────┐
│  Flake Detection Agent  │  Uses AI-SDK ToolLoopAgent
│  - Watches test runs    │  to identify flaky tests
│  - Pattern analysis     │
└───────────┬─────────────┘
            │ flake reports
            ▼
┌─────────────────────────┐
│  Evaluation Framework    │  Measures precision/recall
│  - Compare vs ground    │  against known flaky tests
│  - Benchmark strategies │
└─────────────────────────┘
```

### Components

**1. Test Run Simulator**
- Generates synthetic Vitest test runs in JSON format
- Configurable flake rates and patterns
- Tracks ground truth (which tests are actually flaky)
- Simulates realistic failure patterns:
  - Timing-dependent (race conditions, timeouts)
  - Resource-dependent (port conflicts, file locks)
  - Order-dependent (test pollution, shared state)
  - Intermittent failures (network, external services)

**2. Flake Detection Agent**
- Long-running process that watches for new test runs
- Uses AI-SDK's ToolLoopAgent for the agent loop
- Analyzes patterns across multiple runs
- Identifies tests that fail intermittently
- Outputs structured flake reports

**3. Evaluation Framework**
- Compares agent's identifications against ground truth
- Measures precision, recall, F1 score
- Benchmarks different prompting strategies
- Tracks false positives and false negatives

## Project Structure

Monorepo with three packages:

```
flake-agent/
├── packages/
│   ├── simulator/     # Test run generator
│   ├── agent/          # Flake detection agent
│   └── eval/           # Evaluation framework
└── docs/
    └── adr/            # Architecture decision records
```

Each package is an independent npm workspace with its own dependencies and tests.

## Technology Choices (Initial)

These are exploratory choices for this proof-of-concept:

- **TypeScript** - Matches production constraints
- **npm workspaces** - Monorepo management
- **Vitest JSON reporter format** - Structured test output for programmatic analysis
- **AI-SDK** - Vercel's TypeScript toolkit for building AI agents
- **Node.js** - Runtime environment

Subject to change as we learn what works best.

## Next Steps

1. **Set up project structure** - Initialize TypeScript project with dependencies
2. **Build Test Run Simulator** - Generate synthetic Vitest JSON output with configurable flakes
3. **Implement basic agent** - AI-SDK ToolLoopAgent that analyzes test runs
4. **Create evaluation framework** - Measure agent performance against ground truth
5. **Iterate on prompting** - Experiment with different strategies for flake detection
6. **Add CI integration** - Run evaluation suite as GitHub Actions workflow
