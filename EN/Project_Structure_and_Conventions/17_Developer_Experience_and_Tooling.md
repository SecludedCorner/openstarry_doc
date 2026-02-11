# 17. Developer Experience and Tooling

To enable developers to quickly build, test, and share Agents and plugins, we provide a comprehensive CLI toolchain and development environment.

## 1. OpenStarry CLI (`openstarry`)

`openstarry` serves as the single entry point for both users and developers. For foundational command definitions, refer to **[09. CLI Design and Management Commands](./09_CLI_Design_and_Management_Commands.md)**.

### Common Development Commands (Alias / Extensions)
*   `openstarry init <name>`: Create a new Agent project (includes `agent.json`, `config/`, `.env`).
*   `openstarry create-plugin <name>`: Create a new plugin project from a template.
*   `openstarry dev`: Start local development mode (Hot Reload).
*   `openstarry plugin add`: Install dependency plugins.
*   `openstarry start`: Start the Agent.

## 2. Plugin Scaffolding

Running `openstarry create-plugin my-tool` generates the following standard structure:

```text
my-tool/
├── src/
│   ├── index.ts        # Entry point
│   └── tools.ts        # Tool logic
├── tests/
│   └── index.test.ts   # Pre-configured unit tests
├── package.json        # Dependency configuration
├── tsconfig.json       # TypeScript configuration
└── README.md           # Documentation template
```

**Template Features:**
*   **Pre-configured Build:** Built-in `tsup` or `rollup` configuration for one-command bundling.
*   **Type Safety:** Automatically imports `@openstarry/sdk` type definitions.
*   **Linting:** Pre-configured ESLint/Prettier rules.

## 3. MockHost: The Lab Environment

When developing plugins, spinning up a full Agent (including LLM connections) is both expensive and slow. We provide the `MockHost` testing environment.

### Features
*   **Simulated Core Behavior:** Developers can simulate commands issued by the Core (e.g., `callTool`) within test scripts.
*   **Virtual Context:** No real LLM required -- inject mock Context data directly.
*   **Fast Feedback:** Supports Watch mode for immediate test re-execution upon code changes.

### Usage Example (In Plugin Tests)

```typescript
import { TestAgentHost } from '@openstarry/sdk/testing';
import { MyPlugin } from '../src';

test('MyPlugin loads correctly', async () => {
  const host = new TestAgentHost();
  const plugin = new MyPlugin();

  await host.load(plugin);

  // Simulate Core calling a tool
  const result = await host.executeTool('my_tool_function', { arg: 'value' });

  expect(result).toBe('success');
});
```

## 4. Debugging and Visualization

*   **Inspector:** When `openstarry dev` starts, it opens a local WebSocket port. Developers can connect using `OpenStarry DevTools` (Web UI) to observe the Agent's reasoning process, Context changes, and tool invocation logs in real time.
*   **VS Code Extension:** (Planned for the future) Syntax highlighting for Agent DSL directly in the editor, along with plugin code completion.
