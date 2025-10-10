# package.json

<details><summary>Click to expand..</summary>
 
<br><br>

## General Informations
- https://docs.npmjs.com/creating-a-package-json-file


## Options

<details><summary>Click to expand..</summary>

 
### 1) Identity & Discoverability
- name
  - What: Package identifier; part of URL, CLI, folder.
  - Guidance: lowercase, URL-safe; scoped for orgs.
  - Security: Avoid core module names; immutable branding.
  - Example:
```json
{ "name": "@myorg/eslint-plugin-enterprise" }
```
- version
  - What: Semver; unique with name.
  - Guidance: Follow SemVer rigorously; CI gates on tag vs manifest.
  - Example:
```json
{ "version": "1.4.0" }
```
- description
  - What: Short, clear package summary.
  - Guidance: Helps discovery and internal catalogs.
  - Example:
```json
{ "description": "Enterprise ESLint configurations and rules" }
```
- keywords
  - What: Search tags.
  - Guidance: Add 3–8 relevant terms.
  - Example:
```json
{ "keywords": ["eslint", "eslint-plugin", "enterprise", "security"] }
```
- homepage
  - What: Project landing page.
  - Example:
```json
{ "homepage": "https://github.com/myorg/eslint-plugin-enterprise#readme" }
```
- bugs
  - What: Issue tracker URL and/or email.
  - Example:
```json
{ "bugs": { "url": "https://github.com/myorg/eslint-plugin-enterprise/issues" } }
```
- license
  - What: SPDX expression.
  - Security: Use OSI-approved; for private, use UNLICENSED + private: true.
  - Example:
```json
{ "license": "MIT" }
```
- author, contributors
  - What: People metadata.
  - Guidance: Keep structured; automate via AUTHORS if preferred.
  - Example:
```json
{
  "author": "Jane Doe <jane@corp.com>",
  "contributors": [{ "name": "John Smith", "email": "john@corp.com" }]
}
```
- funding [OPTIONAL]
  - What: Funding links.
  - Example:
```json
{ "funding": "https://corp.com/fund" }
```

### 2) Repository & Support
- repository
  - What: VCS coordinates; supports shortcuts and subdir.
  - Example:
```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/myorg/enterprise-configs.git",
    "directory": "packages/eslint-plugin-enterprise"
  }
}
```

### 3) Entrypoints & Module System
- type
  - What: Module resolution mode.
  - Guidance: Prefer "module" for ESM; use exports map for dual.
  - Example:
```json
{ "type": "module" }
```
- exports 🔐 📦
  - What: Public entrypoints map; controls import/require/types.
  - Security: Prevents private path imports; define subpaths.
  - Example (dual ESM/CJS + types):
```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.cjs",
      "default": "./dist/index.mjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.mjs",
      "require": "./dist/utils.cjs"
    }
  }
}
```


- sideEffects 📦
  - What: Tree-shaking hint.
  - Guidance: false unless files have side effects.
  - Example:
```json
{ "sideEffects": false }
```

### 5) Directories & Files
- files 🔐
  - What: Allow-list published files.
  - Security: Prevents leaking configs/examples/secrets.
  - Example:
```json
{ "files": ["dist", "README.md", "LICENSE"] }
```
- .npmignore interplay
  - Use sparingly; prefer files allow-list.
- directories.*
  - Optional metadata; not widely used by tooling.
  - Example:
```json
{ "directories": { "lib": "dist", "doc": "docs" } }
```

### 6) Scripts & Config
- scripts
  - Guidance: Keep explicit, minimal; avoid postinstall in libs; use prepare in libs to build before publish; in apps, scripts are free but harden via pnpm policy.
  - Example (pnpm-friendly):
```json
{
  "scripts": {
    "build": "tsup",
    "changeset": "changeset",
    "clean": "rimraf dist",
    "commit": "pnpm dlx @commitlint/prompt-cli",
    "commitlint": "commitlint",
    "commitlint:ci": "commitlint --from=$COMMITLINT_FROM --to=$COMMITLINT_TO --strict --color --format markdown",
    "commitlint:print-config": "commitlint --print-config json | jq .",
    "coverage": "vitest run --coverage",
    "dev": "tsup --watch --config tsup.config.ts",
    "format": "node --experimental-strip-types ./node_modules/prettier/bin/prettier.cjs --config prettier.config.ts -w .",
    "format:check": "node --experimental-strip-types ./node_modules/prettier/bin/prettier.cjs --config prettier.config.ts -c .",
    "preinstall": "npx only-allow pnpm",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "pack:dry": "npm pack --dry-run",
    "prepare": "husky",
    "release": "changeset publish",
    "smoke:pack": "pnpm build && pnpm pack:dry && pnpm dlx publint && pnpm dlx arethetypeswrong --pack",
    "test": "vitest",
    "test:integration": "vitest run --project integration",
    "test:regression": "vitest run --project regression",
    "test:unit": "vitest run --project unit",
    "test:update-snapshots": "vitest --update",
    "test:watch": "vitest --watch",
    "typecheck": "tsc -p tsconfig.json --noEmit"
  }
}
```
- config
  - What: Script-time config via env npm_package_config_*.
  - Example:
```json
{
  "config": { "port": "8080" },
  "scripts": { "start": "node server.js --port=$npm_package_config_port" }
}
```

### 7) Dependency Model
- dependencies
  - What: Runtime deps.
  - Guidance: Pin tighter in apps (e.g., ^ only when acceptable); prefer exact in internal workspaces; keep small surface.
  - Example:
```json
{ "dependencies": { "eslint": "^9.0.0" } }
```
- devDependencies
  - What: Build/test tools.
  - Guidance: Keep out of production runtime; in pnpm CI, rely on lockfile for determinism.
  - Example:
```json
{ "devDependencies": { "typescript": "^5.5.0", "vitest": "^2.0.0" } }
```
- peerDependencies 📦
  - What: Host-provided deps (e.g., react, eslint).
  - Guidance: Broad semver ranges; avoid pinning minors unnecessarily.
  - Example:
```json
{ "peerDependencies": { "eslint": "^9", "typescript": ">=5.3 <6" } }
```
- peerDependenciesMeta 📦
  - What: Mark peers as optional when appropriate.
  - Example:
```json
{ "peerDependenciesMeta": { "typescript": { "optional": true } } }
```


### 8) Engines/Runtime & Package Manager
- engines 🔐
  - What: Supported Node, optionally npm/pnpm.
  - Guidance: Pin supported major; enforce with pnpm engineStrict in workspace.
  - Example:
```json
{ "engines": { "node": ">=20.10 <23", "pnpm": ">=9 <10" } }
```
- packageManager 🔐
  - What: Corepack-aware pin of package manager + version.
  - Example:
```json
{ "packageManager": "pnpm@9.12.0" }
```
- devEngines.runtime (pnpm) 🔐
  - What: Locks local runtime via lockfile; ensures scripts run on same Node.
  - Example:
```json
{
  "devEngines": {
    "runtime": { "name": "node", "version": "^22.7.0", "onFail": "download" }
  }
}
```
- executionEnv.nodeVersion (pnpm per-project) 🔐
  - What: Per-project runtime when workspace root pins useNodeVersion.
  - Example:
```json
{ "executionEnv": { "nodeVersion": "22.7.0" } }
```

### 9) Platform Constraints
- os 🔐
  - What: Allowed or disallowed OS.
  - Example:
```json
{ "os": ["darwin", "linux"] }
```
- cpu 🔐
  - What: Allowed or disallowed CPU architectures.
  - Example:
```json
{ "cpu": ["x64", "arm64"] }
```

### 10) Privacy & Publishing Controls
- private 🔐
  - What: Prevent publish.
  - Guidance: 🚀 apps true; 📦 libs false.
  - Example:
```json
{ "private": true }
```
- publishConfig 🔐 📦
  - What: Publish-time overrides.
  - Common fields: access, tag, registry, directory, linkDirectory, executableFiles.
  - Example:
```json
{
  "publishConfig": {
    "access": "public",
    "tag": "next",
    "registry": "https://registry.npmjs.org/",
    "directory": "dist",
    "linkDirectory": true,
    "executableFiles": ["./dist/cli.js"]
  }
}
```
  - Provenance (if supported by your npm): enable via CLI or `"publishConfig": { "provenance": true }` (verify with your registry/policy).

### 11) Workspaces/Monorepos
- workspaces 🏢 ⚠️
  - What: npm workspaces patterns.
  - Guidance: When using pnpm, prefer `pnpm-workspace.yaml` for behavior/policy; use package.json workspaces only if you need npm client compatibility.
  - Example:
```json
{ "workspaces": ["packages/*"] }
```





</details





<br><br>

## Create package.json file of already existing project.
```bash
npm init --yes
```


<br><br>


## Install from package.json
```bash
npm i
```

<br><br>

## Install from package-lock.json
```bash
npm ci
```

<br><br>

## Use local dependencies via path
```javascript
{
  "dependencies": {
    "anyModuleName": "file:../anyModuleName"
  }
}

```

<br><br>

## set/install latest version of dependencies
```javascript
/* method 1* - run npm update after this/
  "dependencies": {
    "express": "*",
    "mongodb": "*",
    "underscore": "*",
    "rjs": "*",
    "jade": "*",
    "async": "*"
  }
  
/* method 2*/
"dependencies":{
    "foo" : ">=1.4.5"
}
```








<br><br><br><br>

# Poperties

<br><br>

## main
- Set a custom file which should be executed in the beginning when you run your app. If you do not define any file then first index.js will be choosed. If index.js does not exist then server.js will be choosed.
```bash
{ "main": "src/index.js" }
```
<br><br>

### execute async function inside of main file
```bash
const main = async () => {
    //..
}

main().then(() => {
    console.log('main() has been called')
})
```


<br><br>

## license
```javascript
// Then include a file named <filename> at the top level of the package.
{ "license" : "SEE LICENSE IN <filename>" }
```

<br><br>

## module
```javscript
"main": "src/index.cjs",
"module": "src/index.mjs",
```
- You can set entry main file for cjs and module file for esm



<br><br><br><br>

## version

<br><br>

#### Install latest patch version
- If you see ~1.0.2 it means to install version 1.0.2 or the latest patch version such as 1.0.4.
```javascript
"request": "~1.0.2"
```

<br><br>

#### Install latest minor or patch version
- If you see ^1.0.2 it means to install version 1.0.2 or the latest minor or patch version such as 1.1.0.
```javascript
"request": "^1.0.2"
```







<br><br><br><br>

## dependencies (https://docs.npmjs.com/cli/v8/configuring-npm/package-json#dependencies)


<br><br>

#### URLs as Dependencies
```javascript
{ "<module-name>", "git+ssh://git@github.com:npm/cli.git#v1.0.27" }
{ "<module-name>", "git+ssh://git@github.com:npm/cli#semver:^5.0" }
{ "<module-name>", "git+https://isaacs@github.com/npm/cli.git" }
{ "<module-name>", "git://github.com/npm/cli.git#v1.0.27" }
```



<br><br>

#### Private Gitlab URLs as Dependencies
- You must create an deploy token for this (https://stackoverflow.com/a/50314604/13025887)
  1. Log in to your GitLab account.
  2. Go to the project you want to create Deploy Tokens for.
  3. Go to Settings > Repository.
  4. Click on Expand on Deploy Tokens section.
  5. Choose a name and optionally an expiry date for the token.
  6. Choose the desired scopes. <= select read_repository
  7. Click on Create deploy token.
  8. Save the deploy token somewhere safe. Once you leave or refresh the page, you won’t be able to access it again.

**Notice that you must use the username that will be generetaed too when you create the deploy token

```javascript
"dependencies": {
    "<module-name>": "git+http://<username>:<token>@url.git#commithash/branch",
}
```


<br><br>

## exports
- Define ESM or CJS
- https://nodejs.org/api/packages.html#exports
- https://nodejs.org/api/packages.html#conditional-exports

```javascript
 "type": "module",
  "exports": {
    "import": "./dist/express-restify-mongoose.js",
    "require": "./dist/cjs/express-restify-mongoose.js"
  }
```

</details>
