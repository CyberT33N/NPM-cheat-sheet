# .npmrc (https://docs.npmjs.com/cli/v7/configuring-npm/npmrc)
- npm gets its config settings from the command line, environment variables, and npmrc files. The npm config command can be used to update and edit the contents of the user and global npmrc files.

<br><br>


## Use specific Node.js & NPM Version
```bash
// package.json
"engines": {
  "node": ">=10.0.0",
  "npm": ">=6.0.0"
}

// .npmrc
engine-strict=true
```



### Do you still need a .npmrc when using pnpm and a pnpm workspace?
- Short answer: Yes—keep a minimal `.npmrc` for registry/auth/proxy only. Put all pnpm behavior and workspace policy in `pnpm-workspace.yaml`.
- Why:
  - pnpm delegates registry/auth and some networking to npm’s config system. That means tokens, CA, proxies, and per-scope registries still live in `.npmrc` (ideally user or CI level, not committed).
  - pnpm‑specific behavior (resolution, hoisting, store, lockfile policy, peer rules, workspace topology) belongs in `pnpm-workspace.yaml`. This keeps secrets out of VCS and moves policy into code-owned infra.

### Legend (used throughout)
- ✅ recommended default
- 🔐 security‑critical
- ⚠️ situational/conditional
- ❌ avoid/anti‑pattern
- 🏢 workspace/monorepo‑only
- 📦 published library‑specific
- 🚀 application/service‑specific
- 🧪 local‑dev convenience
- 🧩 npm/.npmrc‑only (registry/auth/proxy)

---

## .npmrc (registry/auth/proxy scope) 🧩
Keep this file minimal, do not commit secrets. Prefer user‑level or CI‑injected `.npmrc` and environment variables.

- 🔐 ✅ `strict-ssl=true`
  - What: Enforce TLS validation for registry access.
  - Why: Prevents MITM.
  - Example:
    ```
    strict-ssl=true
    ```

- 🔐 ✅ `ca`, `cafile`, `<URL>:cafile`
  - What: Custom CA chain(s) (PEM) or path for enterprise registry.
  - Why: Private PKI/mitm proxies.
  - Example:
    ```
    cafile=/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
    //registry.corp.com/:cafile=/etc/ssl/certs/corp-ca.pem
    ```

- 🔐 ✅ `//<registry>/:_authToken=${NPM_TOKEN}`
  - What: Bearer token per registry via env var.
  - Why: Never commit tokens.
  - Example:
    ```
    //registry.npmjs.org/:_authToken=${NPM_TOKEN}
    ```

- 🔐 ✅ `<URL>:tokenHelper=/absolute/path/to/helper`
  - What: External program returns short‑lived tokens.
  - Why: Strong auth without storing tokens.
  - Example:
    ``` 
    //registry.corp.com:tokenHelper=/usr/local/bin/registry-token
    ```

- ✅ `registry=https://registry.npmjs.org/` (or internal mirror)
  - What: Default registry.
  - Why: Central governance or mirror.
  - Example:
    ```
    registry=https://registry.corp.com/npm/
    @myorg:registry=https://registry.corp.com/npm/
    ```

- ⚠️ `https-proxy`, `http-proxy`, `noproxy`
  - What: Proxy routing.
  - Why: Enterprise networks.
  - Example:
    ```
    https-proxy=https://user:pass@proxy.corp:443
    noproxy=.corp.local
    ```

- ✅ `git-shallow-hosts=github.com gist.github.com gitlab.com bitbucket.org`
  - What: Shallow clones for common hosts.
  - Why: Faster, less data.
  - Example:
    ```
    git-shallow-hosts=["github.com","gitlab.com"]
    ```

- ✅ `maxsockets`, `fetch-retries`, `fetch-retry-factor`, `fetch-retry-mintimeout`, `fetch-retry-maxtimeout`, `fetch-timeout`
  - What: Tune resilience and concurrency.
  - Why: More stable CI under load.
  - Example:
    ```
    maxsockets=64
    fetch-retries=3
    fetch-retry-factor=10
    fetch-retry-mintimeout=10000
    fetch-retry-maxtimeout=60000
    fetch-timeout=60000
    ```

- ❌ Avoid committing any of: `_authToken`, `cert`, `key` inline PEM; prefer env and user/CI scope.

NPM‑only toggles that do not apply to pnpm (or have different names) and should not be relied upon in `.npmrc`:
- ❌ `legacy-peer-deps`, `package-lock`, `package-lock-only`, `fund`, `audit-level`, `workspaces` (npm), `sign-git-tag`, `git-tag-version`, `init-*`—pnpm has its own equivalents or policies. Use `pnpm-workspace.yaml` for pnpm behavior instead.
