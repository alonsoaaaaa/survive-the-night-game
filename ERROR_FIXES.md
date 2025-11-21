# Error Fixes

## MODULE_NOT_FOUND: Cannot find module '/app/index.js'

### Problem
When deploying the project to Railway, specifically the `game-client` package, the deployment fails during the "Starting Container" phase with the following error:
```
Error: Cannot find module '/app/index.js'
```

### Cause
This error occurs because the `game-client` package (`packages/game-client`) is a **library** package, not a standalone application server. It does not have a `start` script in its `package.json`, nor an `index.js` entry point at the root.

When Railway tries to deploy it, it detects a Node.js project. Since no `start` script is defined, it defaults to running `node index.js` (or similar), which fails because the file does not exist.

The `game-client` is intended to be bundled and used by the `website` package (the frontend application).

### Solution
You should not deploy `packages/game-client` as a standalone service. Instead, you should deploy the **`website`** package, which serves the frontend application and includes the game client code.

#### Steps to Fix:

1.  **Update Railway Configuration:**
    *   Go to your Railway project.
    *   Select the service.
    *   Go to **Settings**.
    *   **Root Directory:** Set this to `/` (or leave empty to use the repository root). **Do NOT** set it to `packages/website`.
    *   **Dockerfile Path:** Scroll down to the **Build** section. Set this to `packages/website/Dockerfile`.
        *   *Note: If you don't see "Dockerfile Path", look for a "Build Provider" setting and ensure it is set to **Dockerfile** (not Nixpacks).*
    *   **Build Context:** Ensure this is set to `/` (Root).

    > **Why?** The Dockerfile needs access to the entire repository (including `packages/game-client` and `packages/game-shared`) to build the website. If you set the Root Directory to `packages/website`, it cannot see the other required packages.

2.  **Verify `packages/website` Deployment:**
    *   Railway will now execute the Dockerfile from the root of the repository.
    *   The `COPY packages/website/package.json ...` commands will succeed because the path exists relative to the root.

3.  **If you intended to deploy the Game Server:**
    *   Follow the same pattern:
    *   **Root Directory:** `/`
    *   **Dockerfile Path:** `packages/game-server/Dockerfile`

## Docker Build Error: "failed to calculate checksum ... not found"

### Problem
After changing the Root Directory to `packages/website`, you may encounter:
```
ERROR: failed to build: ... "/packages/website/package.json": not found
```

### Cause
This confirms that the **Build Context** is incorrect. The Dockerfile contains instructions like:
```dockerfile
COPY packages/website/package.json ./packages/website/
```
These paths are relative to the **Repository Root**. When you set the Root Directory to `packages/website` in Railway, Docker looks for `packages/website/packages/website/package.json`, which does not exist.

### Solution
1.  **Verify Root Directory:**
    *   Go to Railway Settings.
    *   Ensure **Root Directory** is exactly `/` (or empty).
    *   If it is set to `packages/website`, **change it to `/`**. This is the most common cause of this error.

2.  **Verify Dockerfile Path:**
    *   Ensure **Dockerfile Path** is `packages/website/Dockerfile`.

3.  **Push Your Changes:**
    *   **Important:** The error log you provided shows a Dockerfile with at least 27 lines, but your local `packages/website/Dockerfile` has only 16 lines.
    *   This means Railway is building an **old or different version** of the code (likely from GitHub).
    *   **You must commit and push your local changes to GitHub** for Railway to see the correct Dockerfile.
    *   Run:
        ```bash
        git add packages/website/Dockerfile
        git commit -m "Fix Dockerfile"
        git push
        ```

4.  **Trigger a New Deployment:**
    *   After pushing, Railway should automatically redeploy. If not, manually trigger a redeploy.


### Summary
*   **Deploy `game-server`** for the backend.

## Build Error: "ENOENT: no such file or directory ... tsconfig.json"

### Problem
During the build process, you may see errors like:
```
TSConfckParseError: parsing /app/packages/game-server/tsconfig.json failed: Error: ENOENT: no such file or directory
```
This happens when building the `website` (which includes `game-client`).

### Cause
The `game-client` package incorrectly depended on `game-server`.
*   `package.json` had `"@survive-the-night/game-server": "*"`
*   `tsconfig.json` had references to `../game-server`

Since the client build (in Docker) might not have access to the server files (or shouldn't need them), this dependency causes the build to fail when it tries to resolve the server's `tsconfig.json`.

### Solution
I have automatically applied the fix to your local files:
1.  **Removed dependency** in `packages/game-client/package.json`.
2.  **Removed references** in `packages/game-client/tsconfig.json`.

**Action Required:**
*   **Push these changes to GitHub** (`git add .`, `git commit`, `git push`).
*   Trigger a new deployment in Railway.
