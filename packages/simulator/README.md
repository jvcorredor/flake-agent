# @flake-agent/simulator

Generates synthetic Vitest JSON test runs with configurable flake patterns and ground truth tracking.

## Purpose

The simulator produces realistic test run data for evaluating the flake detection agent. It generates multiple test runs from a single configuration, introducing flaky behavior according to specified patterns. Each generated batch includes ground truth metadata identifying which tests are configured as flaky.

This enables:

- **Agent evaluation**: Compare agent's flake identifications against known ground truth
- **Pattern testing**: Validate detection strategies against specific failure modes
- **Reproducibility**: Deterministic generation from seed values

## API Contract

### Main Function

```typescript
interface SimulatorConfig {
  testSuite: TestSuiteConfig;
  flakePatterns: FlakePatternConfig[];
  numRuns: number;
  seed?: number;
}

interface SimulatorOutput {
  runs: VitestJsonOutput[];
  groundTruth: Map<string, boolean>;
}

function generateRuns(config: SimulatorConfig): SimulatorOutput;
```

### Types

#### TestSuiteConfig

Defines the baseline test structure (all tests pass by default):

```typescript
interface TestSuiteConfig {
  files: TestFileConfig[];
}

interface TestFileConfig {
  path: string;
  tests: string[];
}
```

#### FlakePatternConfig

Configures which tests are flaky and how they fail:

```typescript
type FlakePatternType =
  | "timing-dependent"
  | "resource-dependent"
  | "order-dependent"
  | "intermittent";

interface FlakePatternConfig {
  testName: string;
  type: FlakePatternType;
  flakeRate: number;
  options?: Record<string, unknown>;
}
```

#### VitestJsonOutput

Full Vitest JSON reporter format (see [ADR-0002](../../docs/adr/0002-vitest-json-format.md)):

```typescript
interface VitestJsonOutput {
  numTotalTestSuites: number;
  numPassedTestSuites: number;
  numFailedTestSuites: number;
  numPendingTestSuites: number;
  numTotalTests: number;
  numPassedTests: number;
  numFailedTests: number;
  numPendingTests: number;
  numTodoTests: number;
  startTime: number;
  success: boolean;
  testResults: TestResult[];
  coverageMap: Record<string, unknown>;
}

interface TestResult {
  assertionResults: AssertionResult[];
  startTime: number;
  endTime: number;
  status: "passed" | "failed" | "skipped";
  message: string;
  name: string;
}

interface AssertionResult {
  ancestorTitles: string[];
  fullName: string;
  status: "passed" | "failed" | "skipped";
  title: string;
  duration: number;
  failureMessages: string[];
  location: { line: number; column: number } | null;
  meta: Record<string, unknown>;
}
```

## Configuration Format

The simulator accepts configuration in JSON or YAML format.

### Example Configuration

```yaml
testSuite:
  files:
    - path: src/auth.test.ts
      tests:
        - login succeeds
        - login fails with bad password
        - logout clears session
    - path: src/api.test.ts
      tests:
        - GET /users returns list
        - POST /users creates user
        - DELETE /users/:id removes user

flakePatterns:
  - testName: login succeeds
    type: timing-dependent
    flakeRate: 0.15
    options:
      timeoutThreshold: 5000

  - testName: GET /users returns list
    type: resource-dependent
    flakeRate: 0.20
    options:
      errorMessage: "ECONNREFUSED 127.0.0.1:3000"

  - testName: POST /users creates user
    type: order-dependent
    flakeRate: 0.30
    options:
      failInPositions: [0, 1]

  - testName: logout clears session
    type: intermittent
    flakeRate: 0.10

numRuns: 100
seed: 42
```

## Flake Patterns

### Timing-Dependent

Tests that fail due to race conditions, timeouts, or timing issues.

**Detection signals:**

- High variance in test duration
- Failures correlate with longer run times
- Failure messages mention timeouts

**Configuration options:**

- `timeoutThreshold`: Duration in ms above which failure probability increases

### Resource-Dependent

Tests that fail due to external resource unavailability (ports, files, services).

**Detection signals:**

- Specific error messages (port conflicts, file locks, connection refused)
- Failures cluster around resource contention

**Configuration options:**

- `errorMessage`: Specific error message to include in failures

### Order-Dependent

Tests that fail when run in certain positions due to test pollution or shared state.

**Detection signals:**

- Failures correlate with position in test run
- Pass/fail pattern depends on execution order
- Often fails early or late in run

**Configuration options:**

- `failInPositions`: Array of position indices where test fails

### Intermittent

Tests that fail randomly with no clear pattern (network issues, external services).

**Detection signals:**

- Random distribution of failures
- No correlation with duration, order, or other factors
- Consistent flake rate across runs

**Configuration options:**

- None (pure random failure based on flakeRate)

## Ground Truth

The ground truth is a mapping of test names to their flaky status:

```typescript
type GroundTruth = Map<string, boolean>;
```

- `true`: Test is configured as flaky
- `false`: Test is stable (always passes)

Ground truth is derived directly from the configuration - any test with a `flakePatterns` entry is marked as flaky.

## Example Output

Given the configuration above, `generateRuns` produces:

```typescript
{
  runs: [
    {
      numTotalTestSuites: 2,
      numPassedTestSuites: 1,
      numFailedTestSuites: 1,
      // ... full Vitest JSON
      testResults: [
        {
          name: "src/auth.test.ts",
          assertionResults: [
            {
              fullName: "login succeeds",
              status: "failed", // timing-dependent flake triggered
              duration: 5200,
              failureMessages: ["Test timeout: exceeded 5000ms"]
            },
            // ... other tests
          ]
        }
      ]
    },
    // ... 99 more runs
  ],
  groundTruth: new Map([
    ["login succeeds", true],
    ["login fails with bad password", false],
    ["logout clears session", true],
    ["GET /users returns list", true],
    ["POST /users creates user", true],
    ["DELETE /users/:id removes user", false]
  ])
}
```

## Development

```bash
npm run build      # Compile TypeScript
npm run dev        # Watch mode
npm run test:run   # Run tests
```

## See Also

- [ADR-0002: Vitest JSON Format](../../docs/adr/0002-vitest-json-format.md)
- [Project README](../../README.md)
