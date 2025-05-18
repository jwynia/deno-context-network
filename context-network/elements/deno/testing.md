# Testing and Benchmarking in Deno

## Purpose
This node describes Deno's built-in testing and benchmarking capabilities, which eliminate the need for external test runners or benchmarking frameworks.

## Classification
- **Domain:** Backend
- **Stability:** Semi-stable
- **Abstraction:** Structural
- **Confidence:** Established

## Content

### Overview

Deno provides first-class support for testing and benchmarking directly within the runtime. The `deno test` and `deno bench` commands allow developers to write and execute tests and benchmarks without external dependencies, simplifying the development workflow and improving the developer experience.

### Testing with `deno test`

#### Purpose and Functionality

The `deno test` command is used to discover and execute test functions defined using the `Deno.test` API. It provides a simple, built-in way to ensure your code behaves as expected.

Key functionalities include:

- **File Discovery**: Automatically finds test files (conventionally ending in `_test.ts`, `_test.js`, etc.) in the current directory and subdirectories.
- **Test Definition**: Tests are defined using the `Deno.test()` function within these files.
- **Execution**: Runs each defined test function, handling asynchronous operations.
- **Reporting**: Provides clear output indicating which tests passed or failed, including error details and test duration.
- **Filtering**: Allows running specific tests based on name or file path.
- **Permissions**: Integrates with Deno's permission system, allowing tests to declare required permissions.

#### Architecture

```mermaid
graph TD
    A[User] --> B(deno test command);
    B --> C{CLI Layer<br/>(cli/tools/test/mod.rs)};
    C --> D[Discover & Load Files];
    D --> E(Deno Runtime);
    E --> F{JS Runtime Layer<br/>(cli/js/40_test.js)};
    F --> G[Register Tests];
    G --> H{Rust Ops Layer<br/>(cli/ops/testing.rs)};
    H --> C; 
    C --> I[Orchestrate Execution];
    I --> H; 
    H --> F; 
    F --> J[Collect Test Results];
    J --> H; 
    H --> C; 
    C --> K[Report Results];
    K --> A;
```

The testing feature involves interaction between the Rust CLI and the Deno JavaScript runtime:

- The Rust CLI (`cli/tools/test/mod.rs`) is responsible for parsing command-line arguments, discovering test files, and orchestrating the test execution process.
- It loads the test files into the Deno runtime.
- The runtime JavaScript (`cli/js/40_test.js`) provides the `Deno.test` API.
- The Rust CLI communicates with the JavaScript runtime via **Ops** (`cli/ops/testing.rs`) to trigger execution of tests and receive results.

#### Defining Tests

Tests are defined using the `Deno.test` function, which accepts either a name and function, or an object with configuration options:

```typescript
// Basic test
Deno.test("hello world", () => {
  const x = 1 + 2;
  assertEquals(x, 3);
});

// Test with configuration
Deno.test({
  name: "hello world",
  fn: () => {
    const x = 1 + 2;
    assertEquals(x, 3);
  },
  ignore: Deno.build.os === "windows",
  only: false,
  sanitizeResources: true,
  sanitizeOps: true,
});
```

#### Test Options

The `Deno.test` function accepts several options to control test behavior:

- **name**: The name of the test (required)
- **fn**: The test function (required)
- **ignore**: If true, the test is skipped
- **only**: If true, only tests with this option set to true will be run
- **sanitizeResources**: Ensure all resources are cleaned up after the test
- **sanitizeOps**: Ensure all ops are completed after the test
- **permissions**: Specify permissions required by the test

#### Assertions

Deno includes a standard library with various assertion functions:

```typescript
import { assertEquals, assertNotEquals, assertStrictEquals } from "https://deno.land/std/assert/mod.ts";

Deno.test("assertions example", () => {
  assertEquals(1, 1);
  assertNotEquals(1, 2);
  assertStrictEquals("hello", "hello");
});
```

Common assertion functions include:

- `assertEquals(actual, expected)`: Assert that values are equal
- `assertNotEquals(actual, expected)`: Assert that values are not equal
- `assertStrictEquals(actual, expected)`: Assert that values are strictly equal
- `assertThrows(fn, ErrorClass)`: Assert that a function throws an error
- `assertRejects(fn, ErrorClass)`: Assert that a promise rejects with an error

#### Test Lifecycle Hooks

While Deno doesn't have built-in lifecycle hooks like `beforeEach` or `afterEach`, they can be implemented using functions:

```typescript
function setupTest() {
  // Set up test environment
  return {
    // Return test context
    cleanup: () => {
      // Clean up test environment
    }
  };
}

Deno.test("test with setup and teardown", () => {
  const { cleanup } = setupTest();
  try {
    // Test code
  } finally {
    cleanup();
  }
});
```

#### Running Tests

To run tests, use the `deno test` command:

```bash
# Run all tests in the current directory and subdirectories
deno test

# Run specific test files
deno test test/user_test.ts test/account_test.ts

# Run tests with a specific name pattern
deno test --filter "user"

# Run tests with specific permissions
deno test --allow-read --allow-net

# Run tests and display only errors
deno test --quiet
```

### Benchmarking with `deno bench`

#### Purpose and Functionality

The `deno bench` command is used to execute benchmark functions defined using the `Deno.bench` API. It measures the performance of specific code snippets by running them repeatedly and calculating metrics like iterations per second or average execution time.

Key functionalities include:

- **File Discovery**: Automatically finds benchmark files (conventionally ending in `_bench.ts`, `_bench.js`, etc.).
- **Benchmark Definition**: Benchmarks are defined using the `Deno.bench()` function.
- **Execution**: Runs each defined benchmark function multiple times to gather performance data.
- **Reporting**: Provides structured output showing benchmark name, number of runs, and performance metrics.
- **Filtering**: Allows running specific benchmarks.

#### Architecture

```mermaid
graph TD
    A[User] --> B(deno bench command);
    B --> C{CLI Layer<br/>(cli/tools/bench/mod.rs)};
    C --> D[Discover & Load Files];
    D --> E(Deno Runtime);
    E --> F{JS Runtime Layer<br/>(cli/js/40_bench.js)};
    F --> G[Register Benchmarks];
    G --> H{Rust Ops Layer<br/>(cli/ops/testing.rs)};
    H --> C;
    C --> I[Orchestrate Execution];
    I --> H;
    H --> F;
    F --> J[Collect Benchmark Results];
    J --> H;
    H --> C;
    C --> K[Report Results];
    K --> A;
```

The architecture for benchmarking is very similar to testing, involving the same core components but orchestrated by a different CLI module.

#### Defining Benchmarks

Benchmarks are defined using the `Deno.bench` function, similar to tests:

```typescript
// Basic benchmark
Deno.bench("string concatenation", () => {
  let s = "";
  for (let i = 0; i < 1000; i++) {
    s += "a";
  }
});

// Benchmark with configuration
Deno.bench({
  name: "array join",
  fn: () => {
    const arr = [];
    for (let i = 0; i < 1000; i++) {
      arr.push("a");
    }
    arr.join("");
  },
  ignore: false,
  baseline: true, // Use this benchmark as a baseline for comparison
  group: "string operations", // Group related benchmarks
});
```

#### Benchmark Options

The `Deno.bench` function accepts several options to control benchmark behavior:

- **name**: The name of the benchmark (required)
- **fn**: The benchmark function (required)
- **ignore**: If true, the benchmark is skipped
- **baseline**: If true, use this benchmark as a baseline for comparison
- **group**: Group related benchmarks together
- **permissions**: Specify permissions required by the benchmark

#### Running Benchmarks

To run benchmarks, use the `deno bench` command:

```bash
# Run all benchmarks in the current directory and subdirectories
deno bench

# Run specific benchmark files
deno bench bench/string_bench.ts bench/array_bench.ts

# Run benchmarks with a specific name pattern
deno bench --filter "string"

# Run benchmarks with specific permissions
deno bench --allow-read --allow-net

# Run benchmarks in a specific group
deno bench --group "string operations"
```

### Common Testing and Benchmarking Patterns

#### Table-Driven Tests

When testing functions with multiple inputs and expected outputs, table-driven tests are efficient:

```typescript
import { assertEquals } from "https://deno.land/std/assert/mod.ts";

function add(a: number, b: number): number {
  return a + b;
}

Deno.test("add function", () => {
  const testCases = [
    { a: 1, b: 2, expected: 3 },
    { a: -1, b: 1, expected: 0 },
    { a: 0, b: 0, expected: 0 },
  ];

  for (const tc of testCases) {
    assertEquals(add(tc.a, tc.b), tc.expected);
  }
});
```

#### Mocking

For unit testing, it's often necessary to mock dependencies:

```typescript
import { assertSpyCalls, spy } from "https://deno.land/std/testing/mock.ts";

Deno.test("spy example", () => {
  const log = spy(console, "log");
  
  console.log("Hello");
  console.log("World");
  
  assertSpyCalls(log, 2);
  log.restore(); // Restore the original method
});
```

#### Stub HTTP Requests

For testing code that makes HTTP requests:

```typescript
Deno.test("http request example", async () => {
  // Set up a simple local server
  const server = Deno.serve({ port: 8000 }, () => {
    return new Response(JSON.stringify({ message: "Hello" }));
  });

  try {
    const response = await fetch("http://localhost:8000");
    const data = await response.json();
    assertEquals(data.message, "Hello");
  } finally {
    server.shutdown();
  }
});
```

### Integration with CI/CD

Deno's testing feature integrates well with CI/CD systems. A typical workflow might include:

```yaml
# .github/workflows/deno.yml
name: Deno CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Setup repo
        uses: actions/checkout@v3

      - name: Setup Deno
        uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x

      - name: Run linter
        run: deno lint

      - name: Run tests
        run: deno test --allow-read

      - name: Run benchmarks
        run: deno bench
```

## Relationships
- **Parent Nodes:** 
  - [overview.md] - is-child-of - The parent overview of Deno
- **Child Nodes:** None
- **Related Nodes:** 
  - [typescript_compilation.md] - depends-on - Testing relies on TypeScript compilation
  - [best_practices.md] - implements - Testing is a key part of best practices
  - [development_workflow.md] - relates-to - Testing and benchmarking are part of the development workflow

## Navigation Guidance
- **Access Context:** Use this node when you need to understand how to test or benchmark Deno applications
- **Common Next Steps:** Explore best_practices.md for guidelines on effective testing or development_workflow.md to understand how testing fits into the overall workflow
- **Related Tasks:** Writing tests for Deno applications, setting up CI/CD with Deno tests, measuring performance with benchmarks
- **Update Patterns:** This node should be updated when there are changes to Deno's testing or benchmarking APIs

## Metadata
- **Created:** 2025-05-18
- **Last Updated:** 2025-05-18
- **Updated By:** AI Assistant

## Change History
- 2025-05-18: Initial creation based on Deno wiki information
