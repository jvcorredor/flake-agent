# ADR-0002: Vitest JSON Reporter Format for Test Output

## Status

Accepted

## Context

The flake detection agent needs to analyze test run output to identify flaky tests. The test run simulator needs to generate realistic test data that the agent can process.

We need a test output format that:
- Is structured and machine-readable
- Contains sufficient detail for analysis (test names, durations, failure messages, stack traces)
- Is realistic enough to validate the agent works on real-world data
- Is easy to generate synthetically

## Decision

Use the **Vitest JSON reporter format** for test run data.

Vitest's JSON reporter outputs structured data including:
- Test suite and test counts (total, passed, failed, pending, todo)
- Test file paths and names
- Test status (passed, failed, skipped)
- Duration per test and per file
- Failure messages and stack traces
- Test hierarchy via `ancestorTitles` (describe blocks)
- Source location (line, column)
- Timestamps for test runs

Example structure (from [Vitest docs](https://vitest.dev/guide/reporters.html#json-reporter)):
```json
{
  "numTotalTestSuites": 4,
  "numPassedTestSuites": 2,
  "numFailedTestSuites": 1,
  "numPendingTestSuites": 1,
  "numTotalTests": 4,
  "numPassedTests": 1,
  "numFailedTests": 1,
  "numPendingTests": 1,
  "numTodoTests": 1,
  "startTime": 1697737019307,
  "success": false,
  "testResults": [
    {
      "assertionResults": [
        {
          "ancestorTitles": ["", "first test file"],
          "fullName": " first test file 2 + 2 should equal 4",
          "status": "failed",
          "title": "2 + 2 should equal 4",
          "duration": 9,
          "failureMessages": ["expected 5 to be 4 // Object.is equality"],
          "location": { "line": 20, "column": 28 },
          "meta": {}
        }
      ],
      "startTime": 1697737019787,
      "endTime": 1697737019797,
      "status": "failed",
      "message": "",
      "name": "/root-directory/__tests__/test-file-1.test.ts"
    }
  ],
  "coverageMap": {}
}
```

## Alternatives Considered

### Jest JSON Format
- **Pros:** Nearly identical to Vitest, widely used
- **Cons:** Slightly different structure, less modern tooling

### Custom Format
- **Pros:** Tailored to our needs
- **Cons:** Not realistic, doesn't validate agent works on real data

### GitHub Actions Annotations
- **Pros:** Native CI format
- **Cons:** Not structured JSON, harder to parse, CI-specific

### TAP (Test Anything Protocol)
- **Pros:** Language-agnostic standard
- **Cons:** Less structured, harder to extract detailed failure info

## Consequences

### Positive
- Structured JSON is easy to parse and generate
- Contains all necessary data for flake analysis (patterns, durations, failures)
- Realistic format validates agent works on real-world data
- Vitest is modern, fast, and well-documented
- Can use Vitest for our own tests (dogfooding)

### Negative
- Tied to Vitest/Jest format (but these are widely used)
- JSON can be verbose for large test suites

### Neutral
- Simulator will generate valid Vitest JSON that could be consumed by any tool expecting this format
- Agent's analysis logic is format-agnostic (could adapt to other formats later)
