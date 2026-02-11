# 16. Plugin Registry and Distribution

OpenStarry utilizes a **"Decentralized Git Repository"** as its core distribution model, rather than relying on a single central server.

## 1. Plugin Distribution Support

### 1.1 The Ecosystem Repo
*   **Repository:** `https://github.com/SecludedCorner/openstarry_plugin`
*   **Role:** The officially maintained "plugin marketplace."
*   **Mechanism:** All standard plugins and audited community plugins are stored in this Monorepo.
*   **Acquisition:** Users clone this repository and keep it updated via `openstarry plugin sync`.

### 1.2 Private/Third-Party Distribution
*   **Mechanism:** Developers can establish their own Git repositories (structure must conform to the `openstarry_plugin` specification).
*   **Acquisition:** After cloning the repository, users perform batch installation using `openstarry plugin add --all`.

### 1.3 NPM-Based Distribution (Recommended)
*   **Mechanism:** The plugin itself is a standard NPM package.
*   **Naming Convention:** `@scope/openstarry-plugin-<name>` or `openstarry-plugin-<name>`.
*   **Advantages:** Leverages existing NPM infrastructure (CDN, version management, dependency resolution).
*   **Installation:** `openstarry plugin add openstarry-plugin-weather`.

### 1.4 Git-Based Distribution (Decentralized)
*   **Mechanism:** Points directly to a Git repository URL.
*   **Scenario:** Internal corporate private plugins or plugins under development.
*   **Installation:** `openstarry plugin add git+https://github.com/user/my-plugin.git`.

## 2. The Registry

While distribution is decentralized, we maintain a **Metadata Registry** for easier discovery.

### Structure
The Daemon maintains an in-memory database (`~/.openstarry/registry.db`) recording:
*   **Plugin ID:** `standard-function-stdio`
*   **Source:** `/Users/me/openstarry_plugin/plugins/standard-function-stdio`
*   **Capabilities:** `[stdio, cli-ui]`
*   **Installed At:** `2026-02-02 10:00:00`
```json
{
  "plugins": {
    "openstarry-plugin-fs": {
      "type": "tool",
      "description": "Safe file system operations",
      "versions": {
        "1.0.0": {
          "url": "npm:openstarry-plugin-fs@1.0.0",
          "checksum": "sha256:..."
        }
      }
    }
  }
}
```
### Query Workflow
When an Agent starts and requests `standard/interaction`, the Core queries the local registry via the Daemon, which returns the physical path of the plugin on the disk.

### 2.2 Submission Workflow
1.  Developer publishes the plugin to NPM.
2.  Developer submits a Pull Request to `openstarry/registry` adding plugin information.
3.  CI automatically validates if the plugin conforms to the `manifest.json` specification.
4.  Upon merging, the plugin becomes searchable via the `openstarry` CLI.

## 3. Installation and Dependency Management

### 3.1 Agent Manifest
The `agent.json` in each Agent instance directory defines its dependencies:

```json
{
  "name": "coding-assistant",
  "plugins": {
    "openstarry-plugin-fs": "^1.2.0",
    "openstarry-plugin-gemini": "latest"
  }
}
```

### 3.2 Installation Process (`openstarry plugin add`)
When a user executes `openstarry plugin add` in the Agent directory:
1.  **Reads** `agent.json`.
2.  **Resolves** dependency sources (NPM or Git).
3.  **Downloads** plugin packages to `~/.openstarry/plugins/` (global cache) or `./node_modules`.
4.  **Verifies** plugin signatures (see Security chapter).
5.  **Generates** a lockfile `openstarry-lock.json` to ensure reproducible environments.

## 4. Plugin Version Control

*   **Official Plugins:** Follow the Git Tag versions of the `openstarry_plugin` repository.
*   **Locking Mechanism:** The project's `openstarry-lock.json` records the current plugin versions (Git Commit Hashes), ensuring execution consistency across different machines.

Adheres to **Semantic Versioning (SemVer)**:
*   **Major:** Breaking changes (e.g., modifying a Tool's parameter structure, causing old Prompts to fail).
*   **Minor:** New features (e.g., adding a new Tool).
*   **Patch:** Bug fixes.

The Agent Core checks the plugin's `engines` field during loading to ensure compatibility with the currently running Core version.
