# 11. Third-Party Plugin Installation

This document defines how the OpenStarry system installs plugins from the official repository (`openstarry_plugin`) or other sources.

## 1. Supported Installation Sources

The OpenStarry installer (`openstarry plugin add`) supports the following three sources:

1.  **Local Path:** A plugin folder you have already downloaded or extracted.
2.  **Git Repository:** A URL pointing directly to GitHub/GitLab.
3.  **NPM Registry:** A standard package published to npm.

## 1.1 Standard Installation Workflow: Repository Sync

This is the **most recommended** way to manage plugins in OpenStarry. The system's standard plugin library is not downloaded item by item; instead, it is acquired by synchronizing with the official ecosystem repository.

### Commands
```bash
# Sync with the official repository (requires openstarry_plugin to be cloned locally first)
$ openstarry plugin sync <path-to-openstarry_plugin-repo>
```

### Execution Logic
1.  **Source Scanning:** The system scans the specified local `openstarry_plugin` repository directory.
2.  **Incremental Update:** Compares versions with those in `~/.openstarry/plugins/` and copies new plugin versions over.
3.  **Dependency Check (Install-time Check):** 
    *   The system immediately checks the `dependencies` list of newly installed plugins.
    *   **If missing:** The CLI immediately outputs a yellow warning:
        > "⚠️ Installed [Workflow], but it requires [Skill]. Please install [Skill] to enable full functionality."
4.  **Automatic Registration:** The Coordination Layer (Daemon) automatically indexes all synchronized plugins.

---

## 2. Developer Mode: Batch Registration (`add --all`)

If you have developed a "Plugin Bundle" containing multiple plugins or a private repository, you can use this command to register them all at once.

### Commands
```bash
# Batch register all plugins in the current directory
$ cd my-private-plugins-repo
$ openstarry plugin add --all .
```

### Execution Logic
1.  **Recursive Scanning:** The system recursively scans the target directory for all subdirectories containing `plugin.json` or `package.json` (with an `openstarry` field).
2.  **Local Linking (Symlink) or Copying:**
    *   By default, the system attempts to create a **Symbolic Link (Symlink)** to `~/.openstarry/plugins/` for easier development and debugging.
    *   Adding the `--copy` parameter performs a physical copy instead.
3.  **Batch Indexing & Verification:** The Daemon updates the index and performs dependency integrity checks on the entire batch of plugins.

## 2.1 The CLI Way

This is the recommended installation method, as the system automatically handles dependencies and paths.

### A. Installing a Local Folder
Assume you have downloaded several plugins into `C:\Downloads\my-plugins`:

```bash
# Syntax: openstarry plugin add <path>
$ openstarry plugin add C:\Downloads\my-plugins\super-search-tool
```

**System Execution Steps:**
1.  **Verification:** Checks the target folder for a valid `package.json` and `plugin.json` (or `openstarry` field).
2.  **Copying:** Copies the folder to the system plugin directory `~/.openstarry/plugins/super-search-tool`.
3.  **Dependencies:** Enters the directory and executes `npm install --production` to install required libraries.
4.  **Building:** If a `tsconfig.json` is detected and no `dist/` directory exists, it attempts `npm run build`.
5.  **Registration:** Notifies the Daemon to refresh the plugin list.

### B. Installing Directly from Git

```bash
$ openstarry plugin add https://github.com/username/weather-plugin.git
```

**System Execution Steps:**
1.  **Clone:** Clones the repository to a temporary directory.
2.  **Processing:** Executes the dependency installation and build steps mentioned above.
3.  **Moving:** Moves the artifacts into the system directory.

---

## 3. The Manual Way

For individual NPM packages or Git URLs, the traditional installation method is still available.

```bash
$ openstarry plugin add https://github.com/user/weather-tool.git
```
*   **Real-time Check:** Upon completion of the installation, the CLI will similarly check and report any missing dependencies.

If you prefer full manual control (e.g., batch copying), please adhere to the following specification:

### Target Path
Place the plugin folder at: `~/.openstarry/plugins/<plugin-id>/`

### Required Operations
Simply copying files is insufficient, as most plugins depend on Node.js modules.

1.  **Copy Files:** 
    Copy the plugin source code to the target path.
    
2.  **Install Dependencies (Crucial Step):** 
    You must enter the directory and install dependencies; otherwise, the Agent will throw an error (`Module not found`) at runtime.
    ```bash
    cd ~/.openstarry/plugins/<plugin-id>
    npm install --production
    ```

3.  **Compilation (If TypeScript):** 
    If the plugin is written in TS and does not include a `dist` folder, you must compile it.
    ```bash
    npm run build
    ```

4.  **Restart Daemon:** 
    ```bash
    openstarry daemon restart
    ```

---

## 4. Plugin Permissions and Isolation

Whether via Sync or Add, when a new plugin is first **Enabled** by an Agent, the system performs an interception check based on the permission settings in `agent.json`, ensuring no unauthorized high-risk operations occur.

When installing a third-party plugin, the system scans the permissions declared in its `manifest`.

*   **Sandboxing:** By default, third-party plugins run in a restricted environment.
*   **Permission Prompts:** When you execute `openstarry plugin add`, the CLI lists the permissions requested by the plugin (e.g., `network`, `fs:write`) and requires your confirmation.
*   **Manual Audit:** For manually installed plugins, the Daemon will intercept the first run and require administrator authorization.
