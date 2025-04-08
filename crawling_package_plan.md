# Plan: Create Enhanced `ragdocs` Server npm Package with Crawling

**Goal:** Create a new, publishable npm package containing an enhanced version of the `ragdocs` MCP server. This version will include multi-page crawling functionality (using a queue-based approach inspired by `hannesrudolph/mcp-ragdocs`) and retain the ability to manage a local Qdrant instance.

**Approach:**
1.  Set up a new Node.js project directory.
2.  Copy the source code from the original `@qpd-v/mcp-server-ragdocs` package into the new project.
3.  Apply modifications for local Qdrant management to the source code.
4.  Implement the new queue-based crawling tools (`extract_urls`, `run_queue`, etc.) within the project structure.
5.  Test thoroughly in a local development setup.
6.  Update documentation within the project.
7.  Prepare the project for npm publishing.
8.  Publish the package to npm (manual user step).

---

## 1. Phase: Project Setup & Initial Code Integration

**Goal:** Prepare a local project structure and integrate the core crawling logic source code.

**Tasks:**

1.1. **Create Project Directory:** Create a new directory for the project (e.g., `mcp-ragdocs-crawler`). Navigate into it.
1.2. **Initialize npm:** Run `npm init -y` to create a basic `package.json`.
1.3. **Copy Source Code:**
    *   Locate the installed `@qpd-v/mcp-server-ragdocs` package source code (likely within `/opt/homebrew/lib/node_modules/@qpd-v/mcp-server-ragdocs/`). *Note: If only `build` output is present, we may need to find the original source repository.*
    *   Create a `src` directory in the new project.
    *   Copy the *source* files (e.g., `.ts` files if it's a TypeScript project, or `.js` files if not) from the original package into the new project's `src` directory, maintaining the original structure.
1.4. **Install Dependencies:** Examine the original package's `package.json` (`dependencies` and `devDependencies`) and install them in the new project using `npm install` or `npm install --save-dev`. Include TypeScript and `@types/node` if it's a TypeScript project.
1.5. **Configure Build:** Set up a build script in the new project's `package.json` (e.g., using `tsc` if TypeScript) to compile `src` into a `build` directory. Ensure necessary configuration files (like `tsconfig.json`) are copied or created.
1.6. **Apply Qdrant Modifications:** Edit the relevant source file (likely `src/index.ts` or equivalent) to add the logic for spawning and managing the local Qdrant process from `~/tools/mcp-ragdocs`, as previously implemented (adding imports, class member, run() logic, cleanup() logic).
1.7. **Create Handler Files:**
    *   Create `src/handlers/` directory.
    *   Create `src/handlers/base-handler.ts` (or `.js`).
    *   Create `src/handlers/extract-urls.ts`: Adapt logic from `/tmp/mcp-ragdocs-clone/build/handlers/extract-urls.js`. Use a project-relative queue path for now (e.g., `./queue.txt`). Convert to TypeScript if necessary.
    *   Create `src/handlers/list-queue.ts`: Implement logic to read `./queue.txt`.
    *   Create `src/handlers/clear-queue.ts`: Implement logic to clear `./queue.txt`.
    *   Create `src/handlers/run-queue.ts`: Adapt logic from `/tmp/mcp-ragdocs-clone/build/handlers/run-queue.js`. Ensure it reads/writes `./queue.txt` and calls the single-page processing logic within the main server class. Convert to TypeScript if necessary.
1.8. **Integrate Handlers in Main File:** Modify the main server source file (e.g., `src/index.ts`):
    *   Import the new handler classes.
    *   Instantiate the new handlers.
    *   Refactor the single-page `add_documentation` logic into a callable method if needed.
    *   Update tool listing logic to include `extract_urls`, `run_queue`, `list_queue`, `clear_queue`.
    *   Update tool calling logic to route to the new handlers.

**Checklist:**

*   [ ] Project directory created, `npm init` run.
*   [ ] Original source code copied into `src/`.
*   [ ] Dependencies installed (`npm install`).
*   [ ] Build process configured and working (`npm run build`).
*   [ ] Local Qdrant management code added to source.
*   [ ] `src/handlers/` directory and handler files created/adapted.
*   [ ] Queue file path set to `./queue.txt` (project relative).
*   [ ] Main server file imports and uses new handlers.
*   [ ] Main server file lists new tools.
*   [ ] Main server file routes calls to new tools.

**Test Procedure:**

1.  **Build Project:** Run `npm run build` in the project directory. Verify it completes without errors and creates the `build` directory.
2.  **Basic Server Start:** Try running the built server directly (`node ./build/index.js`). Check for immediate startup errors related to imports or syntax. (Qdrant connection will likely fail here, which is okay for this step).
3.  **(Post-Implementation Step): Address any issues found during testing.** Fix build errors or immediate runtime errors before proceeding.

---

## 2. Phase: Local Development & Testing

**Goal:** Thoroughly test the integrated crawling functionality using the local project build.

**Tasks:**

2.1. **Configure Roo Client:** Modify your Roo client configuration (`mcp_settings.json` or equivalent) to run the server from the *local project*.
    *   Set the `command` to `node`.
    *   Set the `args` to the *absolute path* of the `build/index.js` file within your new project directory (e.g., `/Users/luca/path/to/mcp-ragdocs-crawler/build/index.js`).
    *   Ensure the `cwd` (current working directory) for the server process is set to your project directory so it can find `./queue.txt`.
2.2. **Restart Roo Client:** Start/Restart the Roo client to connect to the locally running server. Verify it connects and the server starts Qdrant successfully (check logs).
2.3. **Execute Crawling Test Sequence:**
    *   Run `clear_queue`.
    *   Run `extract_urls` (e.g., `https://vuejs.org/guide/introduction`, `add_to_queue: true`).
    *   Run `list_queue`. Verify URLs were added.
    *   Run `run_queue`. Wait for completion.
    *   Run `list_queue`. Verify queue is empty.
    *   Run `list_sources`. Verify new sources exist.
    *   Run `search_documentation` for content from a crawled page. Verify results.
    *   Run `clear_queue`.

**Checklist:**

*   [ ] Roo client configured to run server from local project build.
*   [ ] Server starts via Roo client, including local Qdrant.
*   [ ] `clear_queue` works.
*   [ ] `extract_urls` adds URLs to `./queue.txt`.
*   [ ] `list_queue` shows expected URLs.
*   [ ] `run_queue` processes URLs from `./queue.txt`.
*   [ ] `list_queue` shows empty queue after run.
*   [ ] `list_sources` shows sources added via queue.
*   [ ] `search_documentation` finds content from crawled pages.

**Test Procedure:**

1.  Execute the tool calls sequentially as described in Task 2.3.
2.  Examine tool outputs and the `./queue.txt` file content at each step.
3.  Check server logs via Roo client for any errors during `run_queue`.
4.  **(Post-Implementation Step): Address any issues found during testing.** Debug and fix code in the `src` directory, rebuild (`npm run build`), restart Roo client, and re-test until all steps pass.

---

## 3. Phase: Documentation Update

**Goal:** Create/Update the `README.md` within the new project directory.

**Tasks:**

3.1. **Create/Edit `README.md`:** Write comprehensive documentation in the project's root directory. Include:
    *   Project purpose (Enhanced ragdocs server with crawling).
    *   Setup instructions (Clone repo, `npm install`, Qdrant setup in `~/tools/mcp-ragdocs`, Roo client config pointing to *this* project's build).
    *   Configuration (Environment variables like `EMBEDDING_PROVIDER`, etc.).
    *   Usage: Describe all tools, including the crawling workflow (`extract_urls` -> `run_queue`).
    *   Development instructions (Build process, testing).

**Checklist:**

*   [ ] `README.md` exists in project root.
*   [ ] `README.md` covers purpose, setup, configuration, usage of all tools, crawling workflow, and development.

**Test Procedure:**

1.  **User Review:** User reads the `README.md` and confirms clarity, accuracy, and completeness.
2.  **(Post-Implementation Step): Address any issues found during review.** Update `README.md` based on feedback.

---

## 4. Phase: Prepare for npm Publishing

**Goal:** Finalize the project structure and configuration for publishing.

**Tasks:**

4.1. **Update `package.json`:**
    *   Set `name` (e.g., `@your-npm-username/mcp-ragdocs-crawler`).
    *   Set `version` (e.g., `1.0.0`).
    *   Set `description`.
    *   Set `author`.
    *   Set `repository` URL (pointing to your fork/repo).
    *   Set `license` (e.g., "MIT").
    *   Define `bin` if you want a command-line executable, or ensure `main` points to `build/index.js`.
    *   Define `files` array: `["build", "README.md", "LICENSE"]`.
4.2. **Add `LICENSE` File:** Create a `LICENSE` file in the project root (e.g., containing the MIT License text).
4.3. **Add `.npmignore`:** Create an `.npmignore` file listing files/directories to exclude from the package (e.g., `src/`, `node_modules/`, `tsconfig.json`, `*.log`, `queue.txt`, `.git/`).
4.4. **Final Build:** Run `npm run build` one last time.

**Checklist:**

*   [ ] `package.json`: name, version, description, author, repository, license updated.
*   [ ] `package.json`: `bin` or `main` entry point correctly defined.
*   [ ] `package.json`: `files` array includes `build`, `README.md`, `LICENSE`.
*   [ ] `LICENSE` file created in project root.
*   [ ] `.npmignore` file created and includes appropriate exclusions.
*   [ ] Final `npm run build` successful.

**Test Procedure:**

1.  **Pack Locally:** Run `npm pack` in the project root. This creates a `.tgz` file.
2.  **Inspect Tarball:** Extract the `.tgz` file to a temporary location and inspect its contents. Verify that only the files listed in `package.json`'s `files` array (and implicitly `package.json` itself) are present, and that excluded files/directories are absent.
3.  **(Post-Implementation Step): Address any issues found during testing.** Adjust `.npmignore` or the `files` array in `package.json` and re-pack/re-inspect until correct.

---

## 5. Phase: Publish to npm (Manual User Step)

**Goal:** Make the package publicly available on npm.

**Tasks:**

5.1. **Login to npm:** User runs `npm login` in their terminal and enters credentials.
5.2. **Publish Package:** User runs `npm publish --access public` in the project root directory.

**Checklist:**

*   [ ] User successfully logged into npm via CLI.
*   [ ] `npm publish --access public` command executed without errors.
*   [ ] Package appears on npmjs.com under the specified name.

**Test Procedure:**

1.  **Install Globally:** In a *different* directory or machine, run `npm install -g @your-npm-username/mcp-ragdocs-crawler`. Verify installation completes.
2.  **Configure Roo:** Configure the Roo client to use the globally installed package (similar to the original setup, but using the new package name and ensuring the path from `npm root -g` is correct).
3.  **Basic Test:** Start Roo client, verify the server connects and starts Qdrant. Run `list_tools` and a simple `add_documentation` call.
4.  **(Post-Implementation Step): Address any issues found during testing.** If installation or basic function fails, it may require unpublishing (`npm unpublish <package-name> --force`), fixing the code/configuration in the project, incrementing the version in `package.json`, and re-publishing.

---

This revised plan focuses on building a publishable package from the start. Please review this updated plan.