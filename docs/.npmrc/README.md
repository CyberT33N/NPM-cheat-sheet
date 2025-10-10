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

```
# -----------------------------
# NPM Configuration for Project
# -----------------------------

# 1️⃣ Fund-Option
# Mit "false" werden die Sponsor-/Funding-Hinweise beim npm install unterdrückt.
# Warum? Sauberere Logs in CI/CD und beim Entwickeln.
fund=false  # ✅ Behalten: reduziert Log-Noise; setze auf true, wenn erwünscht

# 2️⃣ Provenance
# Mit "true" wird die Provenance-Funktion aktiviert.
# npm kann nachvollziehen, woher das Paket kommt und ob es unverändert ist.
# Besonders relevant für CI/CD Releases mit OIDC.
provenance=true  # ✅ Behalten: für npm publish mit OIDC; erfordert Registry-Support

# 3️⃣ TLS & Registry-Sicherheit
# strict-ssl
# Zweck: Erzwingt TLS-Zertifikatsprüfung für Registry-Aufrufe (MITM-Schutz).
# Wirkung: Install/Fetch schlägt fehl, wenn Zertifikatskette ungültig ist.
# Beispiel: In CI hinter Corporate-Proxy ggf. eigenes CA-Bundle via `cafile` setzen.
strict-ssl=true

# 4️⃣ Git-Shallow-Clones
# git-shallow-hosts
# Zweck: Für gelistete Hosts nur benötigte Commits fetchen → schneller, weniger Bandbreite.
# Wirkung: Beschleunigt Git-Dependencies von GitHub/GitLab.
# Beispiel: Installation aus git+https://github.com/... nutzt shallow clone.
git-shallow-hosts=["github.com","gitlab.com"]

# 5️⃣ HTTP-Fetch-Robustheit & Parallelität
# maxsockets
# Zweck: Maximale gleichzeitige Verbindungen pro Origin erhöhen.
# Wirkung: Höhere Parallelität bei vielen Tarball/Metadata-Requests.
maxsockets=64

# fetch-retries
# Zweck: Anzahl der automatischen Wiederholungen bei Netzwerkfehlern.
# Wirkung: Stabilere CI-Läufe bei transienten Registry/Netzwerk-Problemen.
fetch-retries=3

# fetch-retry-factor
# Zweck: Exponentieller Backoff-Faktor zwischen Retries.
# Wirkung: Progressiv längere Wartezeit bei wiederholten Fehlern.
fetch-retry-factor=10

# fetch-retry-mintimeout
# Zweck: Minimale Wartezeit (ms) vor dem ersten Retry.
# Beispiel: 10000 ms = 10s.
fetch-retry-mintimeout=10000

# fetch-retry-maxtimeout
# Zweck: Maximale Wartezeit (ms), um übermäßige Delays zu verhindern.
# Beispiel: 60000 ms = 60s.
fetch-retry-maxtimeout=60000

# fetch-timeout
# Zweck: Harte Obergrenze pro HTTP-Request (ms).
# Beispiel: 60000 ms = 60s; balanciert Stabilität und Durchsatz.
fetch-timeout=60000

# 6️⃣ Registry & Auth (Vorlagen – auskommentiert lassen, Secrets nicht committen)
# registry / @scope:registry
# Zweck: Unternehmens-Registry/Mirror konfigurieren; Scopes auf interne Registry lenken.
# Beispiel:
# registry=https://registry.corp.com/npm/
# @myorg:registry=https://registry.corp.com/npm/
#
# Auth-Token per ENV (empfohlen) oder Token-Helper (rotierende Tokens).
# Beispiel (ENV):
# //registry.corp.com/:_authToken=${NPM_TOKEN}
# Beispiel (tokenHelper – absoluter Pfad, keine Args):
# //registry.corp.com:tokenHelper=/usr/local/bin/registry-token
#
# Corporate CA (falls TLS-Inspection/Private PKI):
# cafile=/etc/ssl/certs/corp-root-ca.pem
# oder mehrere per ca[]=...
# ca[]="-----BEGIN CERTIFICATE-----\n...\n-----END CERTIFICATE-----"

# 7️⃣ Proxy (nur falls Unternehmens-Proxy nötig)
# https-proxy=https://user:pass@proxy.corp:443
# proxy=http://user:pass@proxy.corp:8080
# noproxy=.corp.local,localhost,127.0.0.1

# Hinweis:
# Diese Datei gehört ins Repo und gilt lokal & in CI.
# Passe ggf. projekt- oder workflow-spezifisch an.
# Geheimnisse/Tokens niemals committen; Auth per ENV (z. B. NPM_TOKEN) oder User-.npmrc.

```
