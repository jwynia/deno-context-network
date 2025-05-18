# Deno Frameworks and Libraries

## Purpose
This node provides an overview of available frameworks, libraries, and tools in the Deno ecosystem to help developers make informed decisions when building applications.

## Classification
- **Domain:** Backend
- **Stability:** Dynamic
- **Abstraction:** Structural
- **Confidence:** Evolving

## Content

### Overview

The Deno ecosystem is growing rapidly, with numerous frameworks and libraries emerging to support various use cases. This document outlines the most notable frameworks and libraries available for Deno development, organized by category.

### Web Frameworks

#### Fresh

[Fresh](https://fresh.deno.dev/) is a full-stack web framework designed by the Deno team, focusing on speed and simplicity.

**Key Features:**
- Zero runtime overhead with server-side rendering (SSR)
- Island-based client hydration for minimal JavaScript
- File-system based routing
- Preact for UI components
- Tailwind CSS for styling
- TypeScript support
- Hot module reloading
- No build step

**Example:**
```typescript
// routes/index.tsx
import { Handlers, PageProps } from "$fresh/server.ts";

export const handler: Handlers = {
  async GET(req, ctx) {
    const resp = await fetch("https://api.github.com/users/denoland");
    const user = await resp.json();
    return ctx.render(user);
  },
};

export default function Home({ data }: PageProps) {
  return (
    <div>
      <h1>Hello, {data.name}!</h1>
      <p>Bio: {data.bio}</p>
    </div>
  );
}
```

#### Oak

[Oak](https://deno.land/x/oak) is a middleware framework for Deno's HTTP server, similar to Express.js for Node.js.

**Key Features:**
- Middleware architecture
- Router for URL handling
- Context-based request/response handling
- Body parsing
- Static file serving
- Flexible API similar to Koa

**Example:**
```typescript
import { Application, Router } from "https://deno.land/x/oak/mod.ts";

const app = new Application();
const router = new Router();

router.get("/", (ctx) => {
  ctx.response.body = "Hello World!";
});

router.get("/users/:id", (ctx) => {
  ctx.response.body = `User ID: ${ctx.params.id}`;
});

app.use(router.routes());
app.use(router.allowedMethods());

await app.listen({ port: 8000 });
```

#### Aleph.js

[Aleph.js](https://alephjs.org/) is a full-stack framework inspired by Next.js.

**Key Features:**
- React framework
- File-system routing
- SSR and SSG
- Hot module replacement
- ESM imports
- TypeScript support
- CSS/CSS modules

**Example:**
```typescript
// pages/index.tsx
import React from 'react'

export default function Home() {
  return (
    <div>
      <h1>Hello Aleph.js</h1>
      <p>Welcome to use Aleph.js!</p>
    </div>
  )
}
```

#### Opine

[Opine](https://deno.land/x/opine) is an Express-like middleware framework for Deno.

**Key Features:**
- Express-like API
- Middleware system
- Routing
- Template rendering
- File uploads
- Static file serving

**Example:**
```typescript
import { opine } from "https://deno.land/x/opine/mod.ts";

const app = opine();

app.get("/", (req, res) => {
  res.send("Hello World");
});

app.listen(3000);
```

### API Frameworks

#### Hono

[Hono](https://hono.dev/) is a small, simple, and ultrafast web framework for Deno, focusing on APIs.

**Key Features:**
- Ultrafast performance
- Middleware system
- TypeScript support
- Compatible with various runtimes (Deno, Bun, Cloudflare Workers)
- Route validation
- Built-in utilities like JSX support

**Example:**
```typescript
import { Hono } from 'https://deno.land/x/hono/mod.ts'

const app = new Hono()

app.get('/', (c) => c.text('Hello Hono!'))
app.get('/user/:id', (c) => {
  const id = c.req.param('id')
  return c.json({ id })
})

Deno.serve(app.fetch)
```

#### Drash

[Drash](https://drash.land/drash) is a REST microframework focusing on developer experience and standards.

**Key Features:**
- Resource-based routing
- Content negotiation
- Built-in middleware
- Response caching
- Request validation

**Example:**
```typescript
import { Drash } from "https://deno.land/x/drash/mod.ts";

class HomeResource extends Drash.Resource {
  public paths = ["/"];
  public GET(request: Drash.Request, response: Drash.Response): void {
    return response.json({ message: "Hello World!" });
  }
}

const server = new Drash.Server({
  resources: [HomeResource],
  hostname: "localhost",
  port: 1447,
  protocol: "http",
});

server.run();
```

### Database Libraries

#### Deno Postgres

[Deno Postgres](https://deno.land/x/postgres) provides a PostgreSQL driver for Deno.

**Key Features:**
- Connection pooling
- Prepared statements
- Transaction support
- TypeScript support
- Simple API

**Example:**
```typescript
import { Client } from "https://deno.land/x/postgres/mod.ts";

const client = new Client({
  user: "postgres",
  database: "test",
  hostname: "localhost",
  port: 5432,
});

await client.connect();

const result = await client.queryArray("SELECT * FROM users");
console.log(result.rows);

await client.end();
```

#### Deno SQLite

[Deno SQLite](https://deno.land/x/sqlite) provides SQLite database access for Deno.

**Key Features:**
- In-memory and file-based databases
- Prepared statements
- Transaction support
- Simple API

**Example:**
```typescript
import { DB } from "https://deno.land/x/sqlite/mod.ts";

const db = new DB("test.db");
db.execute(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT
  )
`);

db.query("INSERT INTO users (name) VALUES (?)", ["John Doe"]);
const users = db.query("SELECT * FROM users");
console.log(users);

db.close();
```

#### Deno MySQL

[Deno MySQL](https://deno.land/x/mysql) provides a MySQL/MariaDB driver for Deno.

**Key Features:**
- Connection pooling
- Prepared statements
- Transaction support

**Example:**
```typescript
import { Client } from "https://deno.land/x/mysql/mod.ts";

const client = await new Client().connect({
  hostname: "127.0.0.1",
  username: "root",
  db: "test",
  password: "password",
});

const users = await client.execute("SELECT * FROM users");
console.log(users.rows);

await client.close();
```

#### Deno MongoDB

[Deno MongoDB](https://deno.land/x/mongo) provides MongoDB database access for Deno.

**Key Features:**
- Connection pooling
- CRUD operations
- Query building
- Aggregation
- TypeScript support

**Example:**
```typescript
import { MongoClient } from "https://deno.land/x/mongo/mod.ts";

const client = new MongoClient();
await client.connect("mongodb://localhost:27017");

const db = client.database("test");
const users = db.collection("users");

await users.insertOne({
  name: "John Doe",
  email: "john@example.com",
});

const allUsers = await users.find().toArray();
console.log(allUsers);
```

### ORM Libraries

#### Deno Nessie

[Nessie](https://deno.land/x/nessie) is a database migration tool for Deno, supporting PostgreSQL, MySQL, and SQLite.

**Key Features:**
- Database schema versioning
- Migration generation
- Rollback support
- Flexible migration files

**Example:**
```typescript
// migrations/20220101_create_users.ts
import { Migration } from "https://deno.land/x/nessie/mod.ts";

export default class extends Migration {
  async up(): Promise<void> {
    await this.client.queryArray(`
      CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);
  }

  async down(): Promise<void> {
    await this.client.queryArray("DROP TABLE users");
  }
}
```

#### Cotton

[Cotton](https://deno.land/x/cotton) is a SQL-focused ORM built for Deno, supporting PostgreSQL, MySQL, and SQLite.

**Key Features:**
- Query building
- Model definition
- Relationship management
- Transaction support
- Migrations

**Example:**
```typescript
import { Cotton, Model } from "https://deno.land/x/cotton/mod.ts";

const db = new Cotton({
  type: "sqlite",
  database: "test.db",
});

class User extends Model {
  static tableName = "users";
  static fields = {
    id: { type: Number, primaryKey: true, autoIncrement: true },
    name: { type: String },
    email: { type: String, unique: true },
  };
}

await db.register(User);
await User.create({ name: "John Doe", email: "john@example.com" });
const users = await User.all();
console.log(users);
```

### Utility Libraries

#### Deno Standard Library

[Deno Standard Library](https://deno.land/std) provides essential utilities and modules for Deno development.

**Key Modules:**
- **http** - HTTP server and utilities
- **fs** - File system utilities
- **path** - Path manipulation utilities
- **testing** - Testing utilities
- **encoding** - Encoding/decoding utilities
- **datetime** - Date and time utilities
- **log** - Logging utilities

**Example:**
```typescript
import { serve } from "https://deno.land/std/http/server.ts";
import { join } from "https://deno.land/std/path/mod.ts";
import { exists } from "https://deno.land/std/fs/mod.ts";

const server = serve({ port: 8000 });
console.log("Server running on http://localhost:8000");

for await (const req of server) {
  const filePath = join("public", req.url.slice(1) || "index.html");
  
  if (await exists(filePath)) {
    const content = await Deno.readFile(filePath);
    req.respond({ body: content });
  } else {
    req.respond({ status: 404, body: "Not Found" });
  }
}
```

#### Zod

[Zod](https://deno.land/x/zod) is a TypeScript-first schema validation library.

**Key Features:**
- Runtime type validation
- Automatic TypeScript type inference
- Composable schemas
- Error handling

**Example:**
```typescript
import { z } from "https://deno.land/x/zod/mod.ts";

const User = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  age: z.number().min(18).optional(),
});

type User = z.infer<typeof User>; // TypeScript type inference

function createUser(data: unknown) {
  const result = User.safeParse(data);
  if (!result.success) {
    console.error(result.error);
    return null;
  }
  
  const user = result.data; // Typed as User
  return user;
}
```

#### Deno JWT

[Deno JWT](https://deno.land/x/djwt) implements JSON Web Tokens for Deno.

**Key Features:**
- JWT creation
- JWT verification
- Support for various algorithms
- TypeScript support

**Example:**
```typescript
import { create, verify } from "https://deno.land/x/djwt/mod.ts";
import { encodeToString } from "https://deno.land/std/encoding/hex.ts";

const key = await crypto.subtle.generateKey(
  { name: "HMAC", hash: "SHA-512" },
  true,
  ["sign", "verify"],
);

const jwt = await create({ alg: "HS512", typ: "JWT" }, { user: "john" }, key);
console.log(jwt);

const payload = await verify(jwt, key);
console.log(payload);
```

#### Cliffy

[Cliffy](https://deno.land/x/cliffy) is a complete command-line framework for Deno.

**Key Features:**
- Command-line argument parsing
- Interactive prompts
- Tables and output formatting
- Progress bars
- Keypress handling

**Example:**
```typescript
import { Command } from "https://deno.land/x/cliffy/command/mod.ts";

await new Command()
  .name("example")
  .version("1.0.0")
  .description("A command line tool")
  .option("-f, --file <file:string>", "Load file")
  .option("-v, --verbose [verbose:boolean]", "Enable verbose mode")
  .action((options) => {
    console.log("Options:", options);
  })
  .parse(Deno.args);
```

### Frontend Libraries

#### Deno DOM

[Deno DOM](https://deno.land/x/deno_dom) provides a DOM implementation for Deno, allowing for HTML parsing and manipulation.

**Key Features:**
- HTML parsing
- DOM API implementation
- CSS selector support
- Element creation and manipulation

**Example:**
```typescript
import { DOMParser } from "https://deno.land/x/deno_dom/deno-dom-wasm.ts";

const html = `<div id="root"><p>Hello World</p></div>`;
const document = new DOMParser().parseFromString(html, "text/html");
const root = document.getElementById("root");
const p = root.querySelector("p");

console.log(p.textContent); // "Hello World"
```

#### Twind

[Twind](https://twind.dev/) is a tailwind-in-js solution that allows for Tailwind CSS-like utilities.

**Key Features:**
- Zero runtime CSS generation
- TypeScript support
- Compatible with any framework
- Highly optimized

**Example:**
```typescript
import { setup, tw } from "https://esm.sh/twind";

// Setup Twind
setup();

// Use in HTML
const html = `
  <div class="${tw`bg-white p-4 rounded shadow`}">
    <h1 class="${tw`text-xl font-bold text-gray-800`}">Hello Twind</h1>
    <p class="${tw`mt-2 text-gray-600`}">This is styled with Twind!</p>
  </div>
`;
```

### Testing Libraries

#### Deno Test

Deno's built-in testing framework provides everything needed for basic testing.

**Key Features:**
- Built into Deno
- Assertion library
- Test filtering
- Test isolation
- Coverage reporting

**Example:**
```typescript
import { assertEquals } from "https://deno.land/std/testing/asserts.ts";

function sum(a: number, b: number): number {
  return a + b;
}

Deno.test("sum function", () => {
  assertEquals(sum(1, 2), 3);
  assertEquals(sum(-1, 1), 0);
  assertEquals(sum(0, 0), 0);
});
```

#### Rhum

[Rhum](https://deno.land/x/rhum) is a test framework for Deno with a focus on BDD-style testing.

**Key Features:**
- BDD-style test suites
- Mocking
- Test hooks
- Test skipping
- Custom reporters

**Example:**
```typescript
import { Rhum } from "https://deno.land/x/rhum/mod.ts";

Rhum.testPlan("Math", () => {
  Rhum.testSuite("sum()", () => {
    Rhum.testCase("should add two numbers", () => {
      Rhum.asserts.assertEquals(1 + 2, 3);
    });
    
    Rhum.testCase("should handle negative numbers", () => {
      Rhum.asserts.assertEquals(-1 + 1, 0);
    });
  });
});

Rhum.run();
```

#### SuperOak

[SuperOak](https://deno.land/x/superoak) is a HTTP assertion library for Oak, inspired by SuperTest.

**Key Features:**
- HTTP assertions
- Request building
- Response validation
- Cookie handling
- File uploads

**Example:**
```typescript
import { superoak } from "https://deno.land/x/superoak/mod.ts";
import { Application, Router } from "https://deno.land/x/oak/mod.ts";

const app = new Application();
const router = new Router();

router.get("/users", (ctx) => {
  ctx.response.body = [{ id: 1, name: "John" }];
});

app.use(router.routes());

Deno.test("GET /users", async () => {
  const request = await superoak(app);
  await request.get("/users")
    .expect(200)
    .expect("Content-Type", /json/)
    .expect([{ id: 1, name: "John" }]);
});
```

## Relationships
- **Parent Nodes:** 
  - [overview.md] - is-child-of - The parent overview of Deno
- **Child Nodes:** None
- **Related Nodes:** 
  - [best_practices.md] - complements - Frameworks support best practices
  - [development_workflow.md] - enhances - Frameworks enhance the development workflow

## Navigation Guidance
- **Access Context:** Use this node when you need to select appropriate frameworks or libraries for Deno development
- **Common Next Steps:** Explore best_practices.md for guidance on how to use these frameworks effectively
- **Related Tasks:** Setting up a new Deno project, evaluating frameworks for specific requirements, updating dependencies
- **Update Patterns:** This node should be updated as new frameworks emerge or existing ones evolve

## Metadata
- **Created:** 2025-05-18
- **Last Updated:** 2025-05-18
- **Updated By:** AI Assistant

## Change History
- 2025-05-18: Initial creation based on Deno framework research
