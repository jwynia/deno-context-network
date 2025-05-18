# Deno Best Practices

## Purpose
This node outlines recommended best practices, patterns, and approaches for developing applications with Deno.

## Classification
- **Domain:** Backend
- **Stability:** Semi-stable
- **Abstraction:** Structural
- **Confidence:** Evolving

## Content

### Overview

Deno's architecture and design principles significantly influence how applications should be built. This document outlines recommended patterns and practices for effective development with Deno.

### Code Organization

#### Project Structure

A well-organized Deno project typically follows this structure:

```
my-deno-project/
├── .vscode/            # VS Code configuration
│   └── settings.json   # Editor settings
├── .github/            # GitHub workflows
│   └── workflows/      # CI/CD configurations
├── src/                # Source code
│   ├── deps.ts         # Central dependencies file
│   ├── main.ts         # Entry point
│   ├── routes/         # Route handlers
│   ├── models/         # Data models
│   ├── utils/          # Utility functions
│   └── config.ts       # Configuration
├── tests/              # Test files
│   └── main_test.ts    # Tests for main.ts
├── scripts/            # Helper scripts
│   └── build.ts        # Build script
├── deno.json           # Deno configuration
├── import_map.json     # Import map (optional)
├── LICENSE             # License file
└── README.md           # Project documentation
```

#### Module Patterns

##### The `deps.ts` Pattern

Centralize external dependencies in a `deps.ts` file to maintain consistent versions across your project:

```typescript
// deps.ts
export { serve } from "https://deno.land/std@0.195.0/http/server.ts";
export { assertEquals } from "https://deno.land/std@0.195.0/testing/asserts.ts";
export { z } from "https://deno.land/x/zod@v3.21.4/mod.ts";

// In other files
import { serve, z } from "./deps.ts";
```

Benefits:
- Single place to update versions
- Easier to track dependencies
- Improved load performance
- Simpler dependency management

##### Module Export Patterns

Export modules using a `mod.ts` file as the public API:

```typescript
// mod.ts
export { createUser } from "./user.ts";
export { createProduct } from "./product.ts";
export type { User } from "./types.ts";

// user.ts (internal implementation)
import { db } from "./db.ts";
export function createUser() { /* ... */ }

// External usage
import { createUser } from "https://example.com/my-module/mod.ts";
```

### Security Practices

#### Principle of Least Privilege

Only request the permissions your application actually needs:

```bash
# Bad: Too many permissions
deno run --allow-all app.ts

# Good: Only necessary permissions with appropriate scope
deno run --allow-net=api.example.com --allow-read=./data --allow-env=API_KEY app.ts
```

#### Handling Sensitive Data

Use environment variables for sensitive information:

```typescript
// Load from .env file
import { load } from "https://deno.land/std@0.195.0/dotenv/mod.ts";
await load({ export: true });

// Access environment variables
const apiKey = Deno.env.get("API_KEY");
if (!apiKey) throw new Error("API_KEY environment variable is required");
```

#### Validating User Input

Always validate user input before processing:

```typescript
import { z } from "https://deno.land/x/zod@v3.21.4/mod.ts";

const UserSchema = z.object({
  username: z.string().min(3).max(20),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
});

function createUser(data: unknown) {
  const result = UserSchema.safeParse(data);
  if (!result.success) {
    throw new Error(`Invalid user data: ${result.error}`);
  }
  
  const user = result.data;
  // Process valid user data
}
```

### Performance Optimization

#### Caching Strategies

Leverage Deno's built-in caching:

```typescript
// Manually cache dependencies before deployment
deno cache --lock=lock.json --lock-write deps.ts

// Use lock file in production
deno run --lock=lock.json --cached-only main.ts
```

For application-level caching:

```typescript
// Simple in-memory cache
const cache = new Map<string, { data: any; expiry: number }>();

async function fetchWithCache(url: string, ttlMs = 60000) {
  const now = Date.now();
  const cacheKey = `fetch:${url}`;
  const cached = cache.get(cacheKey);
  
  if (cached && cached.expiry > now) {
    return cached.data;
  }
  
  const response = await fetch(url);
  const data = await response.json();
  
  cache.set(cacheKey, {
    data,
    expiry: now + ttlMs
  });
  
  return data;
}
```

#### Lazy Loading

Use dynamic imports for code that isn't needed immediately:

```typescript
async function generatePdf() {
  // Only load the PDF library when needed
  const { createPdf } = await import("./pdf_utils.ts");
  return createPdf();
}
```

### Error Handling

#### Proper Error Types

Create specific error types for better error handling:

```typescript
class ResourceNotFoundError extends Error {
  constructor(resource: string, id: string) {
    super(`${resource} with ID ${id} not found`);
    this.name = "ResourceNotFoundError";
  }
}

try {
  const user = await getUser(id);
  if (!user) throw new ResourceNotFoundError("User", id);
} catch (error) {
  if (error instanceof ResourceNotFoundError) {
    // Handle not found case
  } else {
    // Handle other errors
  }
}
```

#### Async Error Handling

Use try-catch blocks with async/await for error handling:

```typescript
async function fetchData() {
  try {
    const response = await fetch("https://api.example.com/data");
    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("Failed to fetch data:", error);
    // Rethrow or handle appropriately
    throw error;
  }
}
```

### Testing Strategies

#### Use Test Coverage

Use Deno's built-in test coverage tools:

```bash
deno test --coverage=coverage_dir
deno coverage coverage_dir
```

#### Table-Driven Tests

Use table-driven tests for functions with multiple inputs:

```typescript
import { assertEquals } from "https://deno.land/std/testing/asserts.ts";

function sum(a: number, b: number): number {
  return a + b;
}

Deno.test("sum function", () => {
  const testCases = [
    { a: 1, b: 2, expected: 3 },
    { a: -1, b: 1, expected: 0 },
    { a: 0, b: 0, expected: 0 },
  ];

  for (const { a, b, expected } of testCases) {
    assertEquals(sum(a, b), expected);
  }
});
```

### Web Development

#### API Design with Oak

When building REST APIs with Oak, organize routes by resource:

```typescript
import { Application, Router } from "https://deno.land/x/oak/mod.ts";

const app = new Application();
const router = new Router();

// User routes
router.get("/users", getUsers);
router.get("/users/:id", getUser);
router.post("/users", createUser);
router.put("/users/:id", updateUser);
router.delete("/users/:id", deleteUser);

// Product routes
router.get("/products", getProducts);
// More routes...

app.use(router.routes());
app.use(router.allowedMethods());

await app.listen({ port: 8000 });
```

#### Middleware Pattern

Use middleware for cross-cutting concerns:

```typescript
// Authentication middleware
async function authMiddleware(ctx: Context, next: () => Promise<void>) {
  const token = ctx.request.headers.get("Authorization")?.split(" ")[1];
  if (!token) {
    ctx.response.status = 401;
    ctx.response.body = { error: "Unauthorized" };
    return;
  }
  
  try {
    ctx.state.user = await verifyToken(token);
    await next();
  } catch (error) {
    ctx.response.status = 401;
    ctx.response.body = { error: "Invalid token" };
  }
}

// Apply middleware
router.get("/protected", authMiddleware, protectedHandler);
```

### Deployment Best Practices

#### Containerization

Create a Dockerfile for containerized deployment:

```dockerfile
FROM denoland/deno:1.37.0

WORKDIR /app

# Cache dependencies
COPY deps.ts .
RUN deno cache deps.ts

# Copy source code
COPY . .

# Compile the app (optional but improves startup time)
RUN deno cache main.ts

# Set permissions
CMD ["run", "--allow-net", "--allow-env", "--allow-read=/app", "main.ts"]
```

#### Environment Configuration

Use configuration files for different environments:

```typescript
// config.ts
const env = Deno.env.get("ENVIRONMENT") || "development";

export const config = {
  development: {
    database: {
      host: "localhost",
      port: 5432,
    },
    logLevel: "debug",
  },
  production: {
    database: {
      host: Deno.env.get("DB_HOST") || "db.example.com",
      port: Number(Deno.env.get("DB_PORT")) || 5432,
    },
    logLevel: "info",
  },
}[env];
```

#### Health Checks

Implement health check endpoints for monitoring:

```typescript
router.get("/health", (ctx) => {
  ctx.response.body = {
    status: "healthy",
    timestamp: new Date().toISOString(),
    version: "1.0.0",
  };
});
```

### Documentation

#### Code Documentation

Use JSDoc comments for documentation:

```typescript
/**
 * Creates a new user in the database
 * @param {Object} userData - The user data
 * @param {string} userData.username - The username
 * @param {string} userData.email - The email address
 * @returns {Promise<User>} The created user
 * @throws {ValidationError} If the user data is invalid
 */
export async function createUser(userData: {
  username: string;
  email: string;
}): Promise<User> {
  // Implementation
}
```

Generate documentation with:

```bash
deno doc --html --output=docs src/
```

### Tooling

#### Use deno.json for Configuration

Configure your project using deno.json:

```json
{
  "tasks": {
    "start": "deno run --allow-net --allow-env src/main.ts",
    "test": "deno test --allow-net",
    "fmt": "deno fmt",
    "lint": "deno lint"
  },
  "fmt": {
    "useTabs": false,
    "lineWidth": 80,
    "indentWidth": 2,
    "singleQuote": true,
    "proseWrap": "always"
  },
  "lint": {
    "files": {
      "include": ["src/"]
    },
    "rules": {
      "tags": ["recommended"]
    }
  },
  "importMap": "./import_map.json"
}
```

#### Formatting and Linting

Regularly format and lint your code:

```bash
# Format code
deno fmt

# Lint code
deno lint
```

#### Use GitHub Actions for CI/CD

Implement GitHub Actions for continuous integration:

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
      - uses: actions/checkout@v3
      - uses: denoland/setup-deno@v1
        with:
          deno-version: v1.x
      - name: Format check
        run: deno fmt --check
      - name: Lint
        run: deno lint
      - name: Test
        run: deno task test
```

## Relationships
- **Parent Nodes:** 
  - [overview.md] - is-child-of - The parent overview of Deno
- **Child Nodes:** None
- **Related Nodes:** 
  - [security_model.md] - implements - Best practices implement security principles
  - [module_resolution.md] - applies - Best practices apply module resolution patterns
  - [testing.md] - uses - Best practices incorporate testing strategies
  - [development_workflow.md] - supports - Best practices support the development workflow
  - [frameworks.md] - leverages - Best practices leverage recommended frameworks

## Navigation Guidance
- **Access Context:** Use this node when you need guidance on how to structure and develop Deno applications
- **Common Next Steps:** Explore frameworks.md for specific tools or development_workflow.md for process guidance
- **Related Tasks:** Setting up a new Deno project, optimizing existing Deno code, implementing security best practices
- **Update Patterns:** This node should be updated as the Deno ecosystem evolves and new best practices emerge

## Metadata
- **Created:** 2025-05-18
- **Last Updated:** 2025-05-18
- **Updated By:** AI Assistant

## Change History
- 2025-05-18: Initial creation based on Deno research and best practices
