# RAG Documentation Server - Setup Instructions

This document provides instructions for setting up the `@qpd-v/mcp-server-ragdocs` MCP server to run via the Roo MCP client configuration. The server will utilize a Qdrant database and executable located in `~/tools/mcp-ragdocs`.

**Goal:** Configure the globally installed `ragdocs` server to be managed by Roo, using a local Qdrant instance and storage.

## Prerequisites

*   **Node.js:** Version 18 or higher recommended. Download from [https://nodejs.org/](https://nodejs.org/)
*   **npm:** Usually included with Node.js.
*   **Operating System:** Assumes a Unix-like environment (macOS, Linux) where `~` resolves to the user's home directory.
*   **Roo MCP Client:** An environment where Roo (or a similar MCP client) is installed and configured.
*   **Embedding Provider:**
    *   **Ollama (Default):** Requires Ollama running locally. Download from [https://ollama.com/](https://ollama.com/). Ensure the default embedding model (`nomic-embed-text`) is pulled (`ollama pull nomic-embed-text`).
    *   **OpenAI:** Requires an OpenAI API key.

## Setup Steps

1.  **Install Server Package Globally:**
    Open your terminal and run:
    ```bash
    npm install -g @qpd-v/mcp-server-ragdocs
    ```
    This installs the server scripts into your global `node_modules` directory (location varies, but often `/usr/local/lib/node_modules/` or similar).

2.  **Prepare `~/tools/mcp-ragdocs` Directory:**
    *   Create the necessary directories:
        ```bash
        mkdir -p ~/tools/mcp-ragdocs/storage
        ```
    *   Download the Qdrant v1.13.6 executable for your OS from the links below or the [Qdrant Releases Page](https://github.com/qdrant/qdrant/releases/tag/v1.13.6):
        *   **macOS (Apple Silicon):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-aarch64-apple-darwin.tar.gz`
        *   **macOS (Intel):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-x86_64-apple-darwin.tar.gz`
        *   **Linux (x86_64 GNU):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-x86_64-unknown-linux-gnu.tar.gz`
        *   **Linux (x86_64 MUSL):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-x86_64-unknown-linux-musl.tar.gz`
        *   **Linux (ARM64 MUSL):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-aarch64-unknown-linux-musl.tar.gz`
        *   **Linux (x86_64 AppImage):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-x86_64.AppImage`
        *   **Linux (Debian/Ubuntu amd64):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant_1.13.6-1_amd64.deb` (Install using `dpkg -i`)
        *   **Windows (x86_64):** `https://github.com/qdrant/qdrant/releases/download/v1.13.6/qdrant-x86_64-pc-windows-msvc.zip`
    *   Extract the archive if necessary (e.g., `tar -xzf qdrant-....tar.gz` or unzip). For `.deb`, use `dpkg -x qdrant_....deb temp_dir` and find the executable in `temp_dir/usr/bin/qdrant`. For `.AppImage`, make it executable (`chmod +x`) and run it directly (though placing the underlying executable might be preferable for this setup).
    *   Place the extracted `qdrant` executable file directly inside `~/tools/mcp-ragdocs/`.
    *   Make the Qdrant executable runnable:
        ```bash
        chmod +x ~/tools/mcp-ragdocs/qdrant
        ```
    *   The `~/tools/mcp-ragdocs/storage` directory will be used by Qdrant when the server starts.

3.  **Determine Global Install Path:**
    *   Before modifying scripts or configuring Roo, find the exact global installation path. Run:
        ```bash
        npm root -g
        ```
    *   Note the output path (e.g., `/opt/homebrew/lib/node_modules`). This is your `<global_node_modules_path>`. The server script will be at `<global_node_modules_path>/@qpd-v/mcp-server-ragdocs/build/index.js`.

4.  **Create Qdrant Configuration File:**
    *   Qdrant v1.13.6 uses a configuration file. Create one in the directory prepared earlier:
        ```bash
        # Create the file ~/tools/mcp-ragdocs/config.yaml with the following content:
        cat << EOF > ~/tools/mcp-ragdocs/config.yaml
        storage:
          storage_path: ~/tools/mcp-ragdocs/storage
        EOF
        ```
    *   *(Note: Ensure the path inside `config.yaml` correctly points to the `storage` directory created in step 2, using the full path resolved from `~` if necessary for the tool executing this step).*

5.  **Modify the Globally Installed Server Script:**
    *   The main server script needs modification to find the Qdrant executable, its configuration file, and the storage directory using the correct `~/tools/mcp-ragdocs` location.
    *   **Locate the script:** Use the `<global_node_modules_path>` found in step 3. The script is at `<global_node_modules_path>/@qpd-v/mcp-server-ragdocs/build/index.js`.
    *   **Edit the script:** Open the located `index.js` file in a text editor.
    *   **Apply Changes:**
        *   **Add Imports:** Ensure these imports are present at the top:
            ```javascript
            import path from 'path';
            import { fileURLToPath } from 'url';
            import fs from 'fs';
            import os from 'os';
            ```
        *   **Add Directory Helpers (if missing):** Ensure these lines are present after the imports:
            ```javascript
            const __filename = fileURLToPath(import.meta.url);
            const __dirname = path.dirname(__filename);
            ```
        *   **Modify Qdrant Path (around line 473):** Change the `qdrantPath` definition to use `os.homedir()`:
            ```javascript
            const qdrantPath = path.join(os.homedir(), 'tools', 'mcp-ragdocs', 'qdrant');
            ```
        *   **Define Config and Storage Paths (around line 476):** Ensure `storageDir` uses `os.homedir()` and add a definition for `configPath`:
            ```javascript
            const storageDir = path.join(os.homedir(), 'tools', 'mcp-ragdocs', 'storage');
            const configPath = path.join(os.homedir(), 'tools', 'mcp-ragdocs', 'config.yaml');
            if (!fs.existsSync(storageDir)) {
              console.error(`Creating storage directory: ${storageDir}`);
              fs.mkdirSync(storageDir, { recursive: true });
            }
            ```
        *   **Modify Spawn Command (around line 484):** Change the `spawn` command to use `--config-path` instead of `--storage-path`:
            ```javascript
            const qdrantProcess = spawn(qdrantPath, ['--config-path', configPath], {
                detached: true,
                stdio: 'ignore'
            });
            ```


6.  **Install Server Dependencies (Playwright Browsers & Ollama Model):**
    *   The server uses Playwright to fetch web content and requires browser binaries.
    *   If using the default Ollama provider, the embedding model needs to be pulled.
    *   Run the following commands, using the `<global_node_modules_path>` found in step 3:
        ```bash
        # Navigate to the installed package directory
        cd <global_node_modules_path>/@qpd-v/mcp-server-ragdocs

        # Install Playwright browser binaries
        npx playwright install

        # Pull the default Ollama embedding model (if using Ollama)
        ollama pull nomic-embed-text
        ```
    *   *(Ensure you `cd` back to your original directory if needed after running these commands).* 

7.  **Configure Roo MCP Client:**
    *   Edit your Roo MCP client's configuration file (e.g., `mcp_settings.json`). Common locations include `~/.config/roo/mcp_settings.json` or within VS Code's global storage (`~/Library/Application Support/Code/User/globalStorage/...`). If unsure, you may need to search for the file.
    *   Add a new server entry under `servers`. Use `stdio` as the transport.
    *   **Command:** `node`
    *   **Args:** The full path to the *globally installed and modified* `index.js` script, using the `<global_node_modules_path>` found in step 3 (e.g., `<global_node_modules_path>/@qpd-v/mcp-server-ragdocs/build/index.js`).
    *   **Name:** `ragdocs`
    *   **Environment Variables:** Set any necessary environment variables (`EMBEDDING_PROVIDER`, `OPENAI_API_KEY`, `OLLAMA_URL`) within the server configuration if needed.

    *Example `mcp_settings.json` entry:*
    ```json
    {
      "name": "ragdocs",
      "enabled": true,
      "transport": {
        "type": "stdio",
        "command": "node",
        "args": [
          "<global_node_modules_path>/@qpd-v/mcp-server-ragdocs/build/index.js"
        ],
        "env": {
          // "EMBEDDING_PROVIDER": "openai",
          // "OPENAI_API_KEY": "your-key-here"
        }
      }
    }
    ```

8.  **Start Roo Client:** Launch your Roo MCP client. It should now connect to and manage the `ragdocs` server process according to the configuration.

## Usage

1.  Once the server is running via Roo, you can interact with it using the standard MCP tools (`use_mcp_tool`, `access_mcp_resource`).
2.  **Server Name:** `ragdocs`
3.  **Available Tools:** `add_documentation`, `search_documentation`, `list_sources`.
4.  **Indexing Data:** You need to add data yourself using the `add_documentation` tool for each URL you want to include (e.g., from `vue_docs_links.txt` if you have it).

## Troubleshooting

*   **Server Fails to Start:** Check Roo's logs. Verify the path to `index.js` in `mcp_settings.json` uses the correct `<global_node_modules_path>`. Ensure the script modifications (imports, paths, spawn command with `--config-path`) were applied correctly. Check Qdrant permissions (`chmod +x`). Ensure Playwright browsers were installed (Step 6).
*   **Connection Errors (`ECONNREFUSED`):** Ensure Qdrant started correctly. Check the `~/tools/mcp-ragdocs/config.yaml` file exists and contains the correct `storage_path`. Verify the `index.js` script uses `--config-path` pointing to this file. Try running Qdrant manually (`~/tools/mcp-ragdocs/qdrant --config-path ~/tools/mcp-ragdocs/config.yaml`) to see startup errors.
*   **Embedding Errors / Model Not Found:** Ensure your chosen embedding provider (Ollama/OpenAI) is running and configured correctly (API keys). If using Ollama, ensure the default model (`nomic-embed-text`) was pulled (Step 6).
*   **Playwright Errors (`Executable doesn't exist`):** Ensure the Playwright browsers were installed correctly by running `npx playwright install` within the server's global package directory (Step 6).
