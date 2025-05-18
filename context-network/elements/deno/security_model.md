# Deno Security Model

## Purpose
This node describes Deno's permission-based security model, which is one of its most distinctive features compared to other JavaScript runtimes.

## Classification
- **Domain:** Backend
- **Stability:** Static
- **Abstraction:** Structural
- **Confidence:** Established

## Content

### Overview

Deno operates with a strong emphasis on security, employing a secure-by-default paradigm. By default, Deno programs are executed in a sandboxed environment, restricting access to system resources unless explicitly permitted. This security architecture represents a significant departure from Node.js, which grants programs full system access by default.

### Permission System

The cornerstone of Deno's security model is its granular permission system. Scripts require explicit permissions to:

- Access the file system (`--allow-read`, `--allow-write`)
- Access the network (`--allow-net`)
- Access environment variables (`--allow-env`)
- Run subprocesses (`--allow-run`)
- Access high-resolution time (`--allow-hrtime`)
- Load dynamic libraries (`--allow-ffi`)
- Access system information (`--allow-sys`)

If a script attempts to perform an operation without the necessary permission, it will throw a `Deno.errors.PermissionDenied` error.

### Permission Workflow

```mermaid
graph TD
    A[User via CLI] --> B(cli/args/flags.rs: Parse Permissions Flags)
    B --> C(runtime/permissions.rs: Initialize Permissions Struct)
    C --> D[Deno Runtime (JS/TS Script)]
    D --> E(runtime/ops/permissions.rs: Check Permissions Before Op Execution)
    E -- Granted --> F[System Call (e.g., File Read, Network Access)]
    E -- Denied --> G[Throw PermissionDenied Error]
    F --> H[Resource Accessed]
    G --> D
    H --> D
```

### Permission Granularity

Permissions can be further scoped to specific resources:

```bash
# Allow network access only to example.com and deno.land
deno run --allow-net=example.com,deno.land script.ts

# Allow read access only to the /tmp directory
deno run --allow-read=/tmp script.ts

# Allow write access only to a specific file
deno run --allow-write=/tmp/log.txt script.ts
```

This granularity allows for precise control over what resources a script can access, implementing the principle of least privilege.

### Implementation Details

The security model is implemented across several key files:

#### `cli/args/flags.rs`

This component handles parsing command-line flags related to permissions. It translates user-specified permission flags into internal permission structures.

```rust
// Example from cli/args/flags.rs
#[derive(Debug, Parser)]
#[clap(name = "deno")]
pub struct Flags {
  /// Allow file system read access.
  #[clap(long = "allow-read", value_name = "allow-read")]
  allow_read: Option<Vec<String>>,

  /// Allow network access.
  #[clap(long = "allow-net", value_name = "allow-net")]
  allow_net: Option<Vec<String>>,

  // ... other permission flags
}
```

#### `runtime/permissions.rs`

This file defines the `Permissions` struct, which encapsulates the state of granted permissions for a Deno process. It manages access control for various system resources.

```rust
// Example from runtime/permissions.rs
pub struct Permissions {
  pub read: PermissionState<PathBuf>,
  pub write: PermissionState<PathBuf>,
  pub net: PermissionState<String>,
  pub env: PermissionState<bool>,
  // ... other permissions
}

impl Permissions {
  pub fn check_read(&self, path: &Path) -> Result<(), DenoError> {
    self.read.check(path)
  }

  pub fn check_net(&self, host: &str) -> Result<(), DenoError> {
    self.net.check(host)
  }

  // ... other check methods
}
```

#### `runtime/ops/permissions.rs`

This component contains the Rust "ops" (operations) that are invoked by JavaScript/TypeScript code when performing privileged actions. It checks the `Permissions` struct before allowing access to system resources.

```rust
// Example from runtime/ops/permissions.rs
pub fn op_check_read(
  state: &mut OpState,
  path: String,
) -> Result<(), DenoError> {
  let permissions = state.borrow::<Permissions>();
  permissions.check_read(&PathBuf::from(path))
}
```

### Permission Prompting

In addition to static permission grants, Deno can prompt the user for permissions at runtime when using the `--prompt` flag. This allows for a more interactive security model where users can make decisions about permissions as they're needed.

```bash
deno run --prompt script.ts
```

When a script needs a permission, Deno will display a prompt:

```
⚠️  Deno requests read access to "/users/data.txt". Grant? [g/d (g = grant, d = deny)]
```

### Security Reporting

Deno maintains a security policy in `.github/SECURITY.md` that outlines the procedure for reporting security vulnerabilities within the Deno project. This reflects the project's commitment to security beyond just the runtime itself.

### Best Practices

When developing Deno applications, consider the following security best practices:

1. **Request Only Necessary Permissions**: Only request the permissions your application actually needs.
2. **Scope Permissions Narrowly**: Use the granular permission model to limit access to specific resources.
3. **Document Required Permissions**: Clearly document what permissions your application requires and why.
4. **Consider Permission Prompting**: For interactive applications, consider using the `--prompt` flag to request permissions as needed.
5. **Review Dependencies**: Be aware that imported dependencies will run with the same permissions as your application.

### Code Example: Handling Permission Errors

```typescript
try {
  const data = await Deno.readTextFile("/sensitive/data.txt");
  console.log(data);
} catch (error) {
  if (error instanceof Deno.errors.PermissionDenied) {
    console.error("Permission denied to read the file. Run with --allow-read=/sensitive/data.txt");
  } else {
    console.error("An error occurred:", error.message);
  }
}
```

## Relationships
- **Parent Nodes:** 
  - [overview.md] - is-child-of - The parent overview of Deno
- **Child Nodes:** None
- **Related Nodes:** 
  - [architecture.md] - closely-related - The security model is a core aspect of Deno's architecture
  - [best_practices.md] - implements - Many best practices revolve around proper use of the security model

## Navigation Guidance
- **Access Context:** Use this node when you need to understand Deno's security features or how to properly secure a Deno application
- **Common Next Steps:** Explore best_practices.md for guidelines on how to use the security model effectively
- **Related Tasks:** Configuring permissions for a Deno application, understanding permission errors, implementing proper security measures
- **Update Patterns:** This node should be updated when there are changes to Deno's permission system or security features

## Metadata
- **Created:** 2025-05-18
- **Last Updated:** 2025-05-18
- **Updated By:** AI Assistant

## Change History
- 2025-05-18: Initial creation based on Deno wiki information
