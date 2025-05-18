# Deno Context Network Integration Rules

## Critical Domain Boundary

There are two distinct information domains in Deno projects:

1. **Context Network Domain (Team Memory)**
   - Purpose: Planning, coordination, knowledge preservation
   - Audience: Agents, maintainers, internal team
   - Contains: ALL planning documents, architecture decisions, design discussions

2. **Project Artifact Domain (Build Artifacts)**
   - Purpose: Execution by systems, direct use by end-users
   - Audience: Runtime environments, users, external developers
   - Contains: TypeScript/JavaScript source code, deno.json configuration, import maps, lock files, public documentation

**CRITICAL RULE: NEVER create planning documents, architecture diagrams, or design discussions in the project root or artifact domain. These MUST be placed in the context network.**

## Discovery and Initialization

1. At the start of each task, check for a `.context-network.md` discovery file in the project root.

2. If a `.context-network.md` exists:
   - Read the file to determine the actual location of the context network
   - The location may be specified in a "Location" section
   - The context network could be within the project in a specific directory, in a separate repository, or in a shared location

3. If no `.context-network.md` exists, create one with this structure:
   ```markdown
   # Project Context Network

   ## Location
   The context network for this project is located at: ./context-network/

   ## Purpose
   This context network contains all planning documents, architectural decisions, and team coordination information.

   ## Usage Guidelines
   - ALL planning documents, architecture diagrams, and design discussions MUST be stored within the context network
   - Do NOT place planning or architecture documents in the project root
   - Project files outside the context network should only be files consumed by Deno runtime, build systems, or end users

   ## Navigation
   See the context network's navigation guide at [LOCATION]/discovery.md
   ```

4. Then create the context network structure in the specified location (defaulting to "./context-network/" if nothing else was specified):
   ```
   [LOCATION]/
   ├── discovery.md              # Navigation guide for the network
   ├── foundation/               # Core project information
   │   ├── project_definition.md # Main project purpose and goals
   │   ├── architecture.md       # System architecture overview
   │   └── principles.md         # Guiding principles and standards
   ├── modules/                  # Module-specific information
   │   ├── core/                 # Core module documentation
   │   ├── api/                  # API module documentation
   │   └── utils/                # Utilities module documentation
   ├── processes/                # Process documentation
   │   ├── development.md        # Development workflows
   │   ├── testing.md            # Testing procedures
   │   ├── security.md           # Security and permissions management
   │   └── deployment.md         # Deployment processes
   ├── decisions/                # Architecture and design decisions
   │   ├── decision_001.md       # Individual decision records
   │   └── decision_index.md     # Index of all decisions
   ├── planning/                 # Planning documents
   │   ├── roadmap.md            # Project roadmap
   │   └── milestones.md         # Milestone definitions
   ├── connections/              # Cross-cutting concerns
   │   ├── dependencies.md       # Module dependencies
   │   └── interfaces.md         # Interface definitions
   ├── meta/                     # Information about the network itself
   │   ├── updates.md            # Record of network changes
   │   └── maintenance.md        # Network maintenance procedures
   └── archive/                  # Archived documents from the inbox
   ```

5. Notify the user when creating a new context network.

## Context-Aware Workflow

### Before Task Execution
1. Read the `.context-network.md` discovery file
2. Navigate to the actual context network location it specifies
3. Review the context network's discovery.md navigation guide
4. Load relevant foundation documents into your working context
5. Identify information nodes relevant to your current task
6. Note recent changes documented in meta/updates.md that might impact your work

### During Task Execution
1. Maintain awareness of how your work relates to the established network
2. Reference specific information nodes when making decisions
3. Follow documented navigation protocols within the network
4. Maintain consistency with established terminology and patterns
5. Document conflicts between information nodes when encountered

### After Task Completion
1. Update appropriate information nodes with new content
2. Document new relationships discovered during the task
3. Strengthen existing connections that were validated
4. Update meta/updates.md with your changes
5. For significant changes to the network structure, update the discovery.md navigation guide

## Information Classification System

When classifying information nodes, use these dimension labels:

1. **Domain**: [Primary knowledge area]
   - Deno-specific examples: Runtime, Security, Modules, TypeScript, Testing, Standard Library, Web APIs
   - General examples: Core Concept, Supporting Element, External Factor, Resource, Output

2. **Stability**: [Change frequency expectation]
   - Static: Fundamental principles unlikely to change
   - Semi-stable: Established patterns that evolve gradually
   - Dynamic: Frequently changing information

3. **Abstraction**: [Detail level]
   - Conceptual: High-level ideas and principles
   - Structural: Organizational patterns and frameworks
   - Detailed: Specific implementations and examples

4. **Confidence**: [Information reliability]
   - Established: Verified and reliable information
   - Evolving: Partially validated but subject to refinement
   - Speculative: Exploratory ideas requiring validation

## Standard Information Node Structure

Use this structure for all information nodes:

```markdown
# [Node Title]

## Purpose
[Concise explanation of this node's function]

## Classification
- **Domain:** [Primary knowledge area]
- **Stability:** [Static/Semi-stable/Dynamic]
- **Abstraction:** [Conceptual/Structural/Detailed]
- **Confidence:** [Established/Evolving/Speculative]

## Content
[Primary information organized appropriately]

## Relationships
- **Parent Nodes:** [Broader context nodes]
- **Child Nodes:** [More detailed nodes]
- **Related Nodes:** [Nodes with associative connections]
  - [Node Name] - [Relationship Type] - [Brief description]

## Navigation Guide
- **When to Use:** [Access scenarios]
- **Next Steps:** [Typical navigation paths]
- **Related Tasks:** [Activities where relevant]

## Metadata
- **Created:** [Date]
- **Last Updated:** [Date]
- **Updated By:** [Agent ID/Task]

## Change History
- [Date]: [Brief description of changes]
```

## Relationship Types Reference

Use these standardized relationship types:

1. **Hierarchical Relationships**:
   - `is-parent-of`: Node contains broader context
   - `is-child-of`: Node provides specific details
   - `is-version-of`: Node represents a variant

2. **Associative Relationships**:
   - `relates-to`: General connection
   - `depends-on`: Node requires the target
   - `implements`: Node concretely implements target
   - `extends`: Node builds upon target
   - `contradicts`: Node presents opposing view
   - `complements`: Node works alongside target

3. **Cross-Module Relationships**:
   - `interfaces-with`: Node connects across modules
   - `translates-to`: Equivalent concept in different module
   - `impacts`: Changes to node affect target

4. **Deno-Specific Relationships**:
   - `requires-permission`: Node requires specific Deno permissions
   - `imports-from`: Module import relationship
   - `extends-std`: Extends Deno standard library functionality
   - `provides-api`: Exposes an API for other modules

## Deno Security Considerations

When working with Deno projects, always document security considerations:

1. **Permission Documentation**: For each module, document the exact permissions it requires:
   ```markdown
   ## Required Permissions
   - `--allow-net=api.example.com`: For API communication
   - `--allow-read=./data`: For reading configuration files
   - `--allow-env=API_KEY`: For accessing environment variables
   ```

2. **Permission Scope**: Always specify the narrowest possible permission scope
   - Prefer `--allow-net=example.com` over `--allow-net`
   - Prefer `--allow-read=./config` over `--allow-read`

3. **Security Decisions**: Document all security-related decisions in `decisions/` with clear rationales

4. **Permission Changes**: When a module's permission requirements change, update both:
   - The module documentation in the context network
   - The change history to highlight the security impact

## Deno Module Management

When documenting modules in the context network:

1. **URL Sources**: Document the exact URL sources for external dependencies
   ```markdown
   ## External Dependencies
   - `https://deno.land/std@0.195.0/http/server.ts`: HTTP server functionality
   - `https://deno.land/x/oak@v12.5.0/mod.ts`: Oak middleware framework
   ```

2. **Version Pinning**: Always document the pinned versions of dependencies

3. **Import Maps**: If the project uses import maps, document the mapping structure and rationale

4. **Module Organization**: Document the module organization pattern (e.g., deps.ts pattern)

5. **npm Compatibility**: If using npm packages via npm:, document any compatibility considerations

## Context Window Management

1. When context limitations prevent loading all relevant information:
   - Prioritize task-critical information in your limited context
   - Maintain foundation knowledge while working with specific details
   - Create a multi-step process with explicit transitions
   - Document what information was excluded due to limitations

2. For multi-agent workflows:
   - Include explicit context handoff protocols
   - Document which context elements were used and which were modified
   - Reference specific nodes that future agents should consult

## Change Documentation

After completing a task, update the meta/updates.md file with a changelog entry:

```markdown
## Context Network Update: [Task Name] - [Date]

### Information Nodes Modified
- [Node Name]: [Brief description of changes]
  - **Classification Changes**: [Updates to metadata]
  - **Content Changes**: [Content modifications]
  - **Structure Changes**: [Organization changes]

### New Relationships Established
- [Source Node] → [Relationship Type] → [Target Node]: [Description]

### Relationships Modified
- [Source Node] → [Relationship Type] → [Target Node]: [Description]

### Navigation Implications
- [Task Pattern]: [Navigation path changes]

### Follow-up Recommendations
- [Recommendation]: [Rationale and suggested action]
```

## Verification Process

Before completing a task, verify the following:
1. Bidirectional relationship consistency is maintained
2. Navigation paths remain functional after your changes
3. Modified information maintains appropriate classification
4. Your changes support the network's overall coherence
5. The boundary between context network and project artifacts is preserved:
   - ALL planning documents, architecture diagrams, and design discussions are in the context network
   - ONLY code, configuration, and implementation files are in the project artifact domain
   - NO planning or architecture documents in the project root

## Mode-Specific Guidelines

### Architect/Planning Mode

If you are operating in architect or planning mode:

**CRITICAL WARNING: ALL outputs you create MUST be placed within the context network. NEVER create planning or architecture documents in the project root.**

Follow this path structure:
- Place system designs in foundation/architecture.md or modules/*/architecture.md
- Place implementation plans in planning/
- Place architecture decision records in decisions/
- Place process documentation in processes/

### Implementation Mode

If you are operating in implementation mode:

1. When implementing code, place files in their appropriate project locations
2. When documenting implementation decisions or design patterns, place these in the context network
3. Before implementation, ALWAYS check the context network for relevant design documents
4. Update the context network with any implementation decisions that deviate from the original design
5. Follow Deno best practices:
   - Use the deps.ts pattern for centralizing dependencies
   - Apply the principle of least privilege for permissions
   - Leverage TypeScript for type safety
   - Use URL imports with explicit versioning
   - Structure projects according to Deno conventions
   - Document permission requirements for modules
   - Include tests with each module implementation
