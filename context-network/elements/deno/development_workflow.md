# Deno Development Workflow

## Purpose
This node outlines the recommended development workflow for Deno applications, covering the entire application lifecycle from setup to deployment.

## Classification
- **Domain:** Backend
- **Stability:** Semi-stable
- **Abstraction:** Structural
- **Confidence:** Evolving

## Content

### Overview

A well-defined development workflow is essential for productive Deno application development. This document outlines the recommended approaches for various stages of the development lifecycle, from initial setup to deployment and maintenance.

### Environment Setup

#### Prerequisites

Before starting Deno development, ensure you have the following:

1. **Install Deno**: Follow the official installation instructions for your platform:

   ```sh
   # macOS, Linux using Shell
   curl -fsSL https://deno.land/install.sh | sh

   # Windows using PowerShell
   iwr https://deno.land/install.ps1 -useb | iex

   # Using Homebrew
   brew install deno

   # Using Chocolatey
   choco install deno
   ```

2. **Editor Setup**: Configure your editor with Deno support:

   - **VS Code**: Install the official Deno extension:
     ```json
     // settings.json
     {
       "deno.enable": true,
       "deno.lint": true,
       "deno.unstable": false
     }
     ```

   - **JetBrains IDEs**: Install the official Deno plugin through the marketplace.

   - **Vim/Neovim**: Configure with language servers like `coc-deno` or `lspconfig` with Deno language server.

#### Project Initialization

Initialize a new Deno project with a standard structure:

```bash
# Create project directory
mkdir my-deno-project
cd my-deno-project

# Create basic structure
mkdir -p src tests

# Create configuration files
touch deno.json
touch import_map.json
touch README.md
```

Sample `deno.json` configuration:

```json
{
  "tasks": {
    "start": "deno run --allow-net src/main.ts",
    "dev": "deno run --watch --allow-net src/main.ts",
    "test": "deno test --allow-net tests/",
    "lint": "deno lint",
    "fmt": "deno fmt"
  },
  "lint": {
    "files": {
      "include": ["src/", "tests/"]
    },
    "rules": {
      "tags": ["recommended"]
    }
  },
  "fmt": {
    "options": {
      "useTabs": false,
      "lineWidth": 80,
      "indentWidth": 2,
      "singleQuote": true,
      "proseWrap": "always"
    }
  },
  "compilerOptions": {
    "strict": true
  },
  "importMap": "./import_map.json"
}
```

Sample `import_map.json`:

```json
{
  "imports": {
    "std/": "https://deno.land/std@0.220.0/",
    "@/": "./src/",
    "testing/": "https://deno.land/std@0.220.0/testing/"
  }
}
```

### Local Development

#### Development Server

Deno provides a `--watch` flag for automatic reloading during development:

```bash
# Start development server with auto-reload
deno run --watch --allow-net src/main.ts

# Or use the task from deno.json
deno task dev
```

#### Code Organization

Follow these practices for organizing your Deno codebase:

1. **Centralize dependencies** in a `deps.ts` file:

   ```typescript
   // src/deps.ts
   export { serve } from "std/http/server.ts";
   export { z } from "https://deno.land/x/zod@v3.22.4/mod.ts";
   export type { ServerRequest } from "std/http/server.ts";
   ```

2. **Create modular components** with clear responsibilities:

   ```
   src/
   ├── deps.ts           # Centralized dependencies
   ├── main.ts           # Entry point
   ├── config/           # Configuration
   ├── controllers/      # Request handlers
   ├── models/           # Data models
   ├── services/         # Business logic
   ├── middlewares/      # HTTP middleware
   ├── utils/            # Utility functions
   └── types/            # TypeScript type definitions
   ```

3. **Use named exports** to maintain clear imports:

   ```typescript
   // Bad
   export default function doSomething() {}

   // Good
   export function doSomething() {}
   ```

#### Common Development Tasks

Execute common development tasks using the Deno CLI or tasks from `deno.json`:

```bash
# Format code
deno fmt

# Lint code
deno lint

# Run tests
deno test

# Bundle code (if needed)
deno bundle src/main.ts dist/bundle.js

# Compile to executable (if needed)
deno compile --allow-net src/main.ts
```

### Testing

#### Unit Testing

Write unit tests using Deno's built-in test framework:

```typescript
// tests/utils_test.ts
import { assertEquals } from "testing/asserts.ts";
import { add } from "@/utils/math.ts";

Deno.test("add function", () => {
  assertEquals(add(1, 2), 3);
  assertEquals(add(-1, 1), 0);
  assertEquals(add(0, 0), 0);
});
```

Run tests with:

```bash
deno test
```

#### Integration Testing

For integration tests that require HTTP requests or database access:

```typescript
// tests/api_test.ts
import { assertEquals } from "testing/asserts.ts";
import { app } from "@/main.ts";
import { superoak } from "https://deno.land/x/superoak@4.7.0/mod.ts";

Deno.test("GET /api/users returns users", async () => {
  const request = await superoak(app);
  await request.get("/api/users")
    .expect(200)
    .expect("Content-Type", /json/)
    .expect((res) => {
      assertEquals(Array.isArray(res.body), true);
    });
});
```

#### Code Coverage

Generate coverage reports to identify untested code:

```bash
deno test --coverage=coverage
deno coverage coverage --lcov --output=coverage.lcov
```

For visualization, you can convert the LCOV report to HTML using tools like `lcov-report`:

```bash
# Using npm tool (requires Node.js)
npm install -g lcov-report
lcov-report coverage.lcov
```

### Debugging

#### Console Debugging

Use Deno's logging and debugging features:

```typescript
// Simple logging
console.log("Debug info:", value);

// Structured logging
console.debug({ 
  action: "user_login", 
  userId: user.id, 
  timestamp: new Date() 
});

// Console inspection
console.dir(complexObject, { depth: null, colors: true });
```

#### Using Debugger

Debug Deno applications with the Chrome DevTools:

```bash
# Start with inspector
deno run --inspect-brk src/main.ts

# Start with specific inspector address
deno run --inspect=127.0.0.1:9229 src/main.ts
```

Then open Chrome and navigate to `chrome://inspect` to connect to the Deno process.

#### Error Handling

Implement robust error handling:

```typescript
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof SomeSpecificError) {
    // Handle specific error
    console.error("Specific error:", error.message);
  } else {
    // Handle generic error
    console.error("Unexpected error:", error);
    // Consider error reporting service
  }
}
```

### Dependency Management

#### Managing Dependencies

Deno doesn't have a package manager like npm, but you can manage dependencies effectively:

1. **Lock dependencies** for reproducible builds:

   ```bash
   # Generate a lock file
   deno cache --lock=lock.json --lock-write src/deps.ts

   # Use the lock file when running
   deno run --lock=lock.json src/main.ts
   ```

2. **Vendor dependencies** for offline or controlled environments:

   ```bash
   # Vendor all dependencies
   deno vendor src/deps.ts

   # Run with vendored dependencies
   deno run --import-map=vendor/import_map.json src/main.ts
   ```

#### Updating Dependencies

Update dependencies by changing URLs or using automated tools:

```bash
# Manually update in deps.ts
# Change: export { z } from "https://deno.land/x/zod@v3.21.4/mod.ts";
# To:     export { z } from "https://deno.land/x/zod@v3.22.4/mod.ts";

# Update lock file after changes
deno cache --lock=lock.json --lock-write src/deps.ts
```

### Documentation

#### Code Documentation

Document code using JSDoc comments:

```typescript
/**
 * Authenticates a user with the provided credentials.
 * 
 * @param username - The user's username
 * @param password - The user's password
 * @returns The authenticated user or null if authentication fails
 * @throws {AuthError} If the authentication service is unavailable
 */
export async function authenticate(
  username: string,
  password: string
): Promise<User | null> {
  // Implementation
}
```

Generate documentation:

```bash
deno doc src/ --html --output=docs
```

#### Project Documentation

Maintain comprehensive project documentation:

1. **README.md**: Project overview, installation, and usage instructions
2. **CONTRIBUTING.md**: Guidelines for contributors
3. **API.md**: API documentation for public interfaces
4. **CHANGELOG.md**: Record of changes between versions

### Version Control

#### Git Configuration

Configure Git for Deno projects:

```bash
# Create .gitignore
echo ".DS_Store
.env
.env.local
*.log
coverage/
" > .gitignore
```

#### Branching Strategy

Implement a clear branching strategy:

- `main`: Production-ready code
- `develop`: Integration branch for features
- `feature/name`: Feature development
- `bugfix/name`: Bug fixes
- `release/version`: Release preparation

#### Conventional Commits

Use conventional commit messages for clarity:

```
feat(auth): implement JWT authentication
fix(api): resolve CORS issue in user endpoint
docs: update API documentation
test: add tests for user service
chore: update dependencies
```

### Continuous Integration

#### GitHub Actions

Set up GitHub Actions for automated testing and deployment:

```yaml
# .github/workflows/deno.yml
name: Deno CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

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

      - name: Verify formatting
        run: deno fmt --check

      - name: Run linter
        run: deno lint

      - name: Run tests
        run: deno test --allow-net

      - name: Generate coverage
        run: deno test --coverage=coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          directory: ./coverage/
```

#### Pre-commit Hooks

Use pre-commit hooks to enforce quality:

```bash
# Install pre-commit (requires Python)
pip install pre-commit

# Create .pre-commit-config.yaml
echo "repos:
-   repo: local
    hooks:
    -   id: deno-fmt
        name: Deno Format
        entry: deno fmt
        language: system
        pass_filenames: false
    -   id: deno-lint
        name: Deno Lint
        entry: deno lint
        language: system
        pass_filenames: false
" > .pre-commit-config.yaml

# Install hooks
pre-commit install
```

### Deployment

#### Containerization

Create a Dockerfile for containerized deployment:

```dockerfile
FROM denoland/deno:1.38.0

WORKDIR /app

# Cache dependencies
COPY src/deps.ts .
RUN deno cache deps.ts

# Copy source code
COPY . .

# Compile the main app so that it doesn't need to be compiled each startup
RUN deno cache src/main.ts

# Prefer not to run as root
USER deno

# Set environment variables
ENV PORT=8000

# Run with necessary permissions
CMD ["run", "--allow-net", "--allow-env", "src/main.ts"]
```

Build and run the Docker container:

```bash
docker build -t my-deno-app .
docker run -p 8000:8000 my-deno-app
```

#### Deno Deploy

Deploy to Deno Deploy for serverless execution:

1. Create a `deploy.js` entry point that works with Deno Deploy:

   ```typescript
   // deploy.js
   import { app } from "./src/main.ts";
   
   // Deno Deploy will call this function
   Deno.serve(app.fetch);
   ```

2. Deploy using the Deno Deploy GitHub integration or CLI:

   ```bash
   # Using Deno Deploy CLI
   deno install -A -r https://deno.land/x/deploy/deployctl.ts
   deployctl deploy --project=my-project deploy.js
   ```

#### Traditional Hosting

For traditional hosting environments:

1. **Compile to executable** (if supported by host):

   ```bash
   deno compile --allow-net --allow-env src/main.ts
   ```

2. **Set up process manager** (like PM2):

   ```bash
   # Using PM2 (requires Node.js)
   npm install -g pm2
   pm2 start "deno run --allow-net --allow-env src/main.ts" --name my-deno-app
   ```

### Monitoring and Maintenance

#### Logging

Implement structured logging for production:

```typescript
import * as log from "https://deno.land/std/log/mod.ts";

// Configure logger
await log.setup({
  handlers: {
    console: new log.handlers.ConsoleHandler("DEBUG", {
      formatter: (record) => {
        const { datetime, levelName, msg, args } = record;
        return JSON.stringify({
          time: datetime.toISOString(),
          level: levelName,
          message: msg,
          ...args[0],
        });
      },
    }),
    file: new log.handlers.FileHandler("WARNING", {
      filename: "./logs/app.log",
      formatter: JSON.stringify,
    }),
  },
  loggers: {
    default: {
      level: "DEBUG",
      handlers: ["console", "file"],
    },
  },
});

const logger = log.getLogger();
logger.info("Server started", { port: 8000 });
logger.error("Database connection failed", { error: "Timeout", retries: 3 });
```

#### Health Checks

Implement health check endpoints for monitoring:

```typescript
router.get("/health", (ctx) => {
  const dbHealthy = checkDatabaseConnection();
  const cacheHealthy = checkCacheConnection();
  
  const status = dbHealthy && cacheHealthy ? "healthy" : "unhealthy";
  const statusCode = status === "healthy" ? 200 : 503;
  
  ctx.response.status = statusCode;
  ctx.response.body = {
    status,
    time: new Date().toISOString(),
    services: {
      database: dbHealthy ? "connected" : "disconnected",
      cache: cacheHealthy ? "connected" : "disconnected",
    },
  };
});
```

#### Performance Monitoring

Monitor application performance:

```typescript
// Middleware for request timing
async function timingMiddleware(ctx, next) {
  const start = performance.now();
  await next();
  const ms = performance.now() - start;
  
  ctx.response.headers.set("X-Response-Time", `${ms.toFixed(2)}ms`);
  
  // Log for analytics
  console.log(JSON.stringify({
    path: ctx.request.url.pathname,
    method: ctx.request.method,
    status: ctx.response.status,
    time: ms.toFixed(2),
    timestamp: new Date().toISOString(),
  }));
}
```

### Scaling Considerations

#### Horizontal Scaling

For horizontally scaling Deno applications:

1. **Stateless design**: Ensure your application is stateless or uses external state stores
2. **Load balancing**: Deploy behind a load balancer like Nginx or a cloud load balancer
3. **Connection pooling**: Use connection pools for database and external services

Example Nginx configuration:

```nginx
upstream deno_app {
  server 127.0.0.1:8000;
  server 127.0.0.1:8001;
  server 127.0.0.1:8002;
}

server {
  listen 80;
  server_name example.com;
  
  location / {
    proxy_pass http://deno_app;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }
}
```

#### Worker Threads

Utilize worker threads for CPU-intensive tasks:

```typescript
// main.ts
const worker = new Worker(new URL("./worker.ts", import.meta.url).href, {
  type: "module",
});

worker.postMessage({ task: "processData", data: largeDataset });
worker.onmessage = (e) => {
  console.log("Worker result:", e.data);
};

// worker.ts
self.onmessage = async (e) => {
  const { task, data } = e.data;
  
  if (task === "processData") {
    // CPU-intensive processing
    const result = await processLargeDataset(data);
    self.postMessage(result);
  }
};
```

### Version Updates and Migration

#### Managing Deno Updates

Stay updated with Deno releases:

```bash
# Update Deno
deno upgrade

# Update to specific version
deno upgrade --version 1.37.0

# Check current version
deno --version
```

#### Code Migration

When migrating to newer Deno versions:

1. Read the release notes for breaking changes
2. Update imports to use the latest standard library versions
3. Run tests to identify compatibility issues
4. Consider using feature flags for gradual adoption of new features

## Relationships
- **Parent Nodes:** 
  - [overview.md] - is-child-of - The parent overview of Deno
- **Child Nodes:** None
- **Related Nodes:** 
  - [best_practices.md] - implements - Development workflow implements best practices
  - [testing.md] - incorporates - Development workflow incorporates testing strategies
  - [frameworks.md] - utilizes - Development workflow utilizes frameworks and libraries

## Navigation Guidance
- **Access Context:** Use this node when you need to understand the complete development lifecycle for Deno applications
- **Common Next Steps:** Explore best_practices.md for specific coding practices or frameworks.md for tools to enhance your workflow
- **Related Tasks:** Setting up a new Deno project, implementing CI/CD for Deno, optimizing the development process
- **Update Patterns:** This node should be updated as Deno's tooling and deployment options evolve

## Metadata
- **Created:** 2025-05-18
- **Last Updated:** 2025-05-18
- **Updated By:** AI Assistant

## Change History
- 2025-05-18: Initial creation based on Deno development workflow research
