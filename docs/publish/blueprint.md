## Enterprise‑grade TypeScript NPM Package Blueprint (OSS + Enterprise)

### TL;DR (defaults most large OSS libs converge on)
- ESM‑first, Node LTS targets (Node 20 primary, optionally 22), `type: "module"`.
- Strict SemVer, Conventional Commits, automated releases (Changesets), protected main branch.
- `exports` map with `import` and `types` (add `require` only if truly needed).
- Minimal runtime deps; framework/runtime ties as `peerDependencies` (+ `peerDependenciesMeta`).
- `sideEffects: false`, tree‑shakable API, concise `files` whitelist, sourcemaps on, d.ts bundle shipped.
- pnpm, strict CI with matrix (OS x Node), caching, lint/type/test/build/pack smoke, CodeQL + Scorecard.
- OIDC to npm, `npm publish --provenance`, 2FA enforced on org; Renovate/Dependabot on.
- Husky + lint‑staged for fast local gates; heavy gates (typecheck/test) pre‑push or in CI.
- Solid docs: README, CHANGELOG (auto via Changesets), CONTRIBUTING, CODEOWNERS, SECURITY, LICENSE.

---

## Repository layout (single package)
- `src/` (library source; public API at `src/index.ts`)
- `dist/` (build output; not checked in)
- `tests/` or `test/` (Vitest/Jest; colocated tests also OK)
- `scripts/` (release/maintenance scripts, if any)
- `.github/` (workflows, issue/PR templates, CODEOWNERS)
- Config at root: `tsconfig.json`, `tsup.config.ts`, `eslint.config.js|ts`, `.prettierrc`, `vitest.config.ts`, `.npmrc`, `renovate.json` or `.github/dependabot.yml`
- Policy/docs: `README.md`, `CHANGELOG.md` (generated), `CONTRIBUTING.md`, `SECURITY.md`, `LICENSE`, `CODE_OF_CONDUCT.md`

Monorepo note: prefer pnpm workspaces + Changesets; per‑package `dist/`, shared `tsconfig.base.json`.

---

## package.json (canonical ESM‑first)
```json
{
  "name": "@org/pkg-name",
  "version": "0.0.0",
  "description": "Concise, value-focused description.",
  "type": "module",
  "license": "MIT",
  "author": "Org <oss@org.com>",
  "repository": { "type": "git", "url": "git+https://github.com/org/repo.git" },
  "bugs": { "url": "https://github.com/org/repo/issues" },
  "homepage": "https://github.com/org/repo#readme",
  "engines": { "node": ">=20" },
  "packageManager": "pnpm@9",
  "sideEffects": false,
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./package.json": "./package.json"
  },
  "types": "./dist/index.d.ts",
  "files": [
    "dist/",
    "!dist/**/*.map",
    "!**/*.test.*",
    "!**/*.spec.*",
    "!**/__tests__/"
  ],
  "scripts": {
    "clean": "rimraf dist",
    "build": "tsup",
    "dev": "tsup --watch",
    "typecheck": "tsc -p tsconfig.json --noEmit",
    "lint": "eslint .",
    "format": "prettier -w .",
    "format:check": "prettier -c .",
    "test": "vitest run",
    "test:watch": "vitest",
    "coverage": "vitest run --coverage",
    "pack:dry": "npm pack --dry-run",
    "smoke:pack": "pnpm build && pnpm pack:dry && pnpm dlx publint && pnpm dlx arethetypeswrong --pack",
    "release": "changeset publish",
    "changeset": "changeset",
    "prepare": "husky"
  },
  "peerDependencies": {},
  "peerDependenciesMeta": {},
  "dependencies": {},
  "devDependencies": {
    "typescript": "^5.5.0",
    "tsup": "^8.0.0",
    "vitest": "^2.0.0",
    "eslint": "^9.0.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "prettier": "^3.2.0",
    "husky": "^9.0.0",
    "lint-staged": "^15.0.0",
    "rimraf": "^6.0.0",
    "@changesets/cli": "^2.27.0",
    "publint": "^0.2.8",
    "arethetypeswrong": "^0.16.0"
  },
  "publishConfig": {
    "access": "public",
    "provenance": true
  },
  "funding": "https://github.com/sponsors/org",
  "keywords": ["typescript","node","esm"],
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": ["eslint --fix", "prettier -w"],
    "*.{md,json,yml,yaml}": ["prettier -w"]
  }
}
```

Dual‑module variant (only if needed for `require` consumers):
```json
{
  "type": "module",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  },
  "main": "./dist/index.cjs",
  "types": "./dist/index.d.ts"
}
```

Notes
- Prefer ESM‑only; add CJS only to serve material CJS ecosystems.
- Include `./package.json` in `exports` for tooling.
- `files` whitelist keeps tarball small and clean.
- Put framework/runtime ties (e.g., `react`, `vitest`) in `peerDependencies`; mark optional in `peerDependenciesMeta` when appropriate.

---

## TypeScript config (Node ESM library)
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": false,
    "sourceMap": true,
    "inlineSources": true,
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

Optional: `typesVersions` only if you must support TS < 4.7 (legacy).

---

## Build config (tsup)
```ts
// tsup.config.ts
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['esm'], // add 'cjs' only if dual-publish is required
  dts: true,
  sourcemap: true,
  clean: true,
  target: 'node20',
  treeshake: true,
  minify: false
});
```

---

## Linting & formatting (Flat ESLint + Prettier)
- ESLint (typescript‑eslint v7+), Flat config, Prettier for formatting.
- Enforce in CI and local hooks.
- For libraries, keep rules ergonomic and stable; avoid churny stylistic rules that impede PR velocity.

Minimal flat config example:
```js
// eslint.config.js
import tseslint from 'typescript-eslint';

export default tseslint.config(
  tseslint.configs.recommendedTypeChecked,
  {
    files: ['**/*.ts'],
    languageOptions: { parserOptions: { project: ['./tsconfig.json'] } },
    rules: {
        /*
         * Erzwingt Konsistenz in TypeScript und steuert den Auto-Fixer.
         *    prefer: 'type-imports'          → immer `import type` statt Wert-Import für Typen.
         *    fixStyle: 'separate-type-imports' → separater Top-Level-Block für Typen (kein Inline-Mixing).
         */
        '@typescript-eslint/consistent-type-imports': [
            'error',
            {
                fixStyle: 'separate-type-imports',
                prefer: 'type-imports'
            }
        ],

        /*
         * Verhindert redundante Zuweisungen
         * Import/Export Hygiene (Google/Microsoft Standards)
         *
         *### Warum (Best Practice/Industry Standard)
         *- **Klarer Lesbarkeitsgewinn**: Entfernt No-Op-Assertions und visuelles Rauschen.
         *- **Type-Driven Development**: Erzwingt präzise Typmodelle, Guards und Assertion Functions statt `as`-Workarounds.
         *- **Fehlerprävention**: Verhindert unnötige Non-Null-Assertions (`!`) und tarnt keine defekten Typannahmen.
         *- **Align mit Standards**: Teil von `plugin:@typescript-eslint/recommended-type-checked` (empfohlenes, weit verbreitetes Setup in großen TS-Codebasen)
         */
        '@typescript-eslint/no-unnecessary-type-assertion': [
            'error',
            {
                // "as const" nicht fälschlich flaggen
                checkLiteralConstAssertions: false,
                typesToIgnore: []
            }
        ],
    }
  }
);
```

---

## Git hooks (Husky) and staged checks
Install:
```bash
pnpm dlx husky init
```

Create hooks:
```bash
# .husky/pre-commit
pnpm lint-staged

# .husky/pre-push
pnpm typecheck && pnpm test
```

Rationale
- Pre‑commit: fast, incremental (eslint/prettier); no long‑running tasks.
- Pre‑push: heavier checks (typecheck/test) before code leaves the machine.

---

## GitHub Actions: CI (matrix + cache + strict gates)
```yaml
# .github/workflows/ci.yml
name: CI
on:
  pull_request:
  push:
    branches: [main]
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
permissions:
  contents: read
jobs:
  build-test:
    name: Lint • Typecheck • Test • Build • Pack
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest]
        node: [20, 22]
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: pnpm

      - name: Enable corepack
        run: corepack enable

      - name: Install
        run: pnpm install --frozen-lockfile

      - name: Lint
        run: pnpm lint

      - name: Typecheck
        run: pnpm typecheck

      - name: Test
        run: pnpm coverage

      - name: Build
        run: pnpm build

      - name: Packaging smoke
        run: pnpm smoke:pack

      - name: Upload coverage
        if: runner.os == 'Linux' && matrix.node == 20
        uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/
```

Best practices
- Minimal permissions (job‑level), concurrency to avoid duplicate runs.
- Cache pnpm, freeze lockfile.
- Separate publish from CI; CI must be green before release.

---

## GitHub Actions: Release (Changesets + OIDC + provenance)
```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    branches: [main]
permissions:
  contents: write
  id-token: write  # OIDC for npm provenance
  pull-requests: write
jobs:
  version-or-publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: pnpm
          registry-url: https://registry.npmjs.org

      - name: Enable corepack
        run: corepack enable

      - name: Install
        run: pnpm install --frozen-lockfile

      - name: Create release PR or publish
        uses: changesets/action@v1
        with:
          publish: pnpm release
          commit: "chore(release): version packages"
          title: "chore(release): version packages"
        env:
          NPM_CONFIG_PROVENANCE: "true"
          NPM_CONFIG_FUND: "false"
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Notes
- Use Changesets PR flow; merge triggers publish.
- Prefer OIDC (`id-token: write`) with `--provenance`; only use `NPM_TOKEN` if your org hasn’t enabled OIDC.

---

## Security workflows
CodeQL:
```yaml
# .github/workflows/codeql.yml
name: CodeQL
on:
  schedule: [{ cron: "0 3 * * 1" }]
  pull_request:
  push:
    branches: [main]
permissions:
  security-events: write
  contents: read
jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
        with: { languages: javascript-typescript }
      - uses: github/codeql-action/analyze@v3
```

OpenSSF Scorecard:
```yaml
# .github/workflows/scorecard.yml
name: Scorecard
on:
  schedule: [{ cron: "0 1 * * 1" }]
  push:
    branches: [main]
permissions:
  id-token: write
  security-events: write
  contents: read
jobs:
  analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: ossf/scorecard-action@v2.3.3
        with: { results_file: results.sarif }
      - uses: github/codeql-action/upload-sarif@v3
        with: { sarif_file: results.sarif }
```

Org policy
- Enforce 2FA on org publish, branch protections, required status checks (CI, CodeQL).
- Security disclosure via `SECURITY.md`; dependabot alerts enabled.

---

## Dependency automation
Dependabot (simple) or Renovate (enterprise‑grade policies). Example Dependabot:
```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule: { interval: "weekly" }
    open-pull-requests-limit: 10
    versioning-strategy: increase
```

---

## Docs & governance hygiene
- `README.md`: badges (npm, CI, CodeQL, scorecard), install, usage, API quickstart, support matrix, license.
- `CHANGELOG.md`: generated by Changesets; keep human‑readable headings (Breaking/Features/Fixes).
- `CONTRIBUTING.md`: dev setup, scripts, commit conventions, review/PR checklist.
- `CODEOWNERS`: clear ownership; enables review routing.
- `SECURITY.md`: reporting policy, SLA, supported versions.
- Issue/PR templates: bug report, feature request, PR checklist (tests, types, docs).
- `LICENSE`: OSI‑approved (MIT/BSD‑3/Apache‑2.0 etc.).
- `CODE_OF_CONDUCT.md`: Contributor Covenant commonly used.

---

## Quality gates and local workflow
- Conventional Commits with commitlint (enforced in CI/hook).
- Size budgets only if shipping browser bundles; otherwise pack size + `publint` is sufficient.
- Type tests (optional but excellent): `tsd` or `expect-type`.
- Smoke packaging in CI: `npm pack --dry-run`, `publint`, `arethetypeswrong --pack`.

Commit lint example:
```js
// commitlint.config.cjs
export default { extends: ['@commitlint/config-conventional'] };
```

---

## Publishing policy
- Always build from clean; never publish unbuilt sources.
- `files` whitelist + `sideEffects: false` to maximize tree‑shaking and minimize tarball.
- Tag prereleases with `next` (`changeset pre enter next`) and document upgrade path.
- Don’t run install scripts in published package; no `postinstall` side effects.

`.npmrc` (repo‑local):
```ini
fund=false
provenance=true
```

---

## Optional: CLI support
- Add `bin` with shebang + ESM loader:
```json
{
  "bin": { "pkg-cli": "./dist/cli.js" }
}
```
CLI entry:
```ts
#!/usr/bin/env node
import('node:process'); // keep ESM; no top-level await if Node 20 target is OK
```

---

## Optional: Browser/Edge build
- Only if you truly target browser: add `exports` `browser` condition and a separate tsup build with `platform: 'browser'`, `target: 'es2022'`.
- Avoid polyfills; document peer expectations.

---

## What to include in `files`
- Include: `dist/**`, `LICENSE`, `README.md`, (optionally) `CHANGELOG.md`.
- Exclude: tests, configs, raw sources (unless intentionally shipped), maps (optional).
- npm auto‑includes top‑level README/LICENCE by default; still list in `files` for explicitness.

---

## Monorepo notes (if applicable)
- pnpm workspaces, root `tsconfig.base.json`, per‑package `tsconfig.json`.
- One CI pipeline with matrix; selectively build/test changed packages (turbo/nx optional).
- Changesets for versioning, release PR per batch.

---

## Reference scripts you’ll actually use daily
- `pnpm lint` → `eslint .`
- `pnpm typecheck` → `tsc --noEmit`
- `pnpm test` / `pnpm coverage`
- `pnpm build` → tsup
- `pnpm smoke:pack` → `build + npm pack + publint + arethetypeswrong`
- `pnpm changeset` (create), `pnpm release` (publish)
- Hooks: pre‑commit (lint‑staged), pre‑push (typecheck+test)

---

## Why this matches “top packages” patterns
- Converges on ESM‑first, strict CI, automated SemVer with Changesets, OIDC provenance, minimal tarballs, peer boundary clarity, and fast local feedback loops (husky+lint‑staged).
- Mirrors practices seen across widely‑used OSS libs (TS dominant, flat ESLint, pnpm caching, CodeQL/Scorecard, publish smoke tests).