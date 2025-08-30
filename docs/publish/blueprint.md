## Enterprise‑grade TypeScript NPM Package Blueprint (OSS + Enterprise)

### TL;DR (defaults most large OSS libs converge on)
- ESM‑first, Node LTS targets (Node 20 primary, optionally 22), `type: "module"`.


--  -- 

- Strict SemVer (Semantic Versioning), Conventional Commits, automated releases (Changesets), protected main branch.
time deps; framework/runtime ties as `peerDependencies` (+ `peerDependenciesMeta`).


- `sideEffects: false`, tree‑shakable API, concise `files` whitelist, sourcemaps on, d.ts bundle shipped.


- pnpm, strict CI with matrix (OS x Node), caching, lint/type/test/build/pack smoke, CodeQL + Scorecard.

- OIDC to npm (OpenID Connect verwendet, um sich bei npm zu authentifizieren), `npm publish --provenance`, 2FA enforced on org; Renovate/Dependabot on.

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
     "test": "vitest",
     "test:integration": "vitest run --project integration",
     "test:regression": "vitest run --project regression",
     "test:unit": "vitest run --project unit",
     "test:update-snapshots": "vitest --update",
     "test:watch": "vitest --watch"
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


*sideEffects* heißt:

<details><summary>Click to expand..</summary>

Alles klar, lass uns das **aufschlüsseln**, denn das sind zentrale Best Practices für moderne TypeScript‑/Node‑Pakete:

---

### 1️⃣ `sideEffects: false`

* In **package.json** eingetragen.
* Signalisiert dem **Bundler** (z. B. Webpack, Rollup, Vite):

  > „Wenn ich ein Modul importiere, kann es sicher entfernt werden, wenn es nicht genutzt wird.“
* **Effekt:**

  * **Tree-shaking** funktioniert besser.
  * Unbenutzter Code wird automatisch aus dem finalen Bundle gestrichen.
* **Voraussetzung:** Dein Code darf **keine globalen Nebeneffekte** haben, wie z. B. `console.log()` beim Laden oder Modifikationen von `window`/`global`.

---

### 2️⃣ Tree-shakable API

* Tree-shaking = ungenutzte Exporte werden beim Bundling **automatisch entfernt**.
* Beispiel:

```ts
// utils.ts
export function used() { return 42; }
export function unused() { return 99; }

// consumer.ts
import { used } from './utils';
console.log(used());
```

* Mit Tree-shaking wird `unused()` **nicht ins Bundle gepackt**, spart Bytes und Ladezeit.
* Kombiniert man mit `sideEffects: false`, ist das sehr effektiv.

---

### 3️⃣ Concise `files` whitelist

* **`files`** in package.json: Liste der Dateien, die ins npm-Paket kommen.
* Beispiel aus deinem Blueprint:

```json
"files": [
  "dist/",
  "!dist/**/*.map",
  "!**/*.test.*",
  "!**/*.spec.*",
  "!**/__tests__/"
]
```

* **Zweck:**

  * Paket klein halten
  * Nur relevante Artefakte (z. B. kompilierte `.js` und `.d.ts`) veröffentlichen
  * Testdateien, Source Maps (optional), Rohquellen draußen lassen

---

### 4️⃣ Sourcemaps on

* `sourceMap: true` in tsconfig oder Build-Tool (`tsup`)
* **Sinn:** Debugging erleichtern.
* Beispiel: Du bekommst im Browser/Node die **Original-TypeScript-Zeile** beim Stacktrace, nicht die kompilierte `.js` Zeile.

---

### 5️⃣ d.ts bundle shipped

* `.d.ts` = TypeScript **Deklarationsdatei**.
* Liefert Typdefinitionen für Konsumenten deines Pakets.
* Beispiel: Wenn jemand `import { foo } from 'mypkg'` macht, bekommt er **vollständige Typinformationen**, IntelliSense, Auto-Completion.
* Meistens gebündelt in `dist/index.d.ts`.

---

💡 **Kurz gesagt:**

> Mit diesen Einstellungen lieferst du ein **sauberes, minimal, leicht tree-shakable Paket**, das in TypeScript sauber typisiert ist, schnell lädt und gut debugbar bleibt.


</details>



<br><br>



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

### TypeScript-Config

#### `tsconfig.base.json`
```jsonc
{
    "compilerOptions": {

        /*
         *✅ Erlaubt das Importieren von `.ts`-Dateien mit Erweiterung.
         *- Notwendig für TypeScript ES-Module (`import "./file.ts"`).
         */
        "allowImportingTsExtensions": true,

        /*
         *✅ Erlaubt das Importieren von Modulen, die keinen `default`-Export haben.
         *- Nützlich für CommonJS-Module (`import fs from "fs"`).
         */
        "allowSyntheticDefaultImports": true,

        /*
         *✅ Basisverzeichnis für `paths`-Aliase.
         *- Erlaubt z. B. `import foo from "@/utils/foo"` statt `import foo from "../../utils/foo"`.
         */
        "baseUrl": ".",

        /*
         *✅ Erlaubt Interoperabilität zwischen CommonJS und ES-Modulen.
         *- Falls du CJS (`require()`) und ESM (`import`) mischst, **essentiell**.
         */
        "esModuleInterop": true,

        /*
         *⚠️ **Erzeugt Metadaten für Dekoratoren (z. B. für NestJS, TypeORM).**
         *- **Nur aktiv lassen, wenn du Reflection brauchst.**
         */
        // "emitDecoratorMetadata": true,
        /*
         *✅ Erlaubt experimentelle Dekoratoren (z. B. `@Injectable()` in NestJS).
         *- Dekoratoren sind noch nicht offiziell in JavaScript, daher "experimentell".
         */
        "experimentalDecorators": true,

        /*
         *✅ Erzwingt konsistente Groß-/Kleinschreibung in Imports.
         *- Verhindert Bugs bei case-sensitive Dateisystemen (z. B. Linux).
         */
        "forceConsistentCasingInFileNames": true,

        /*
         *✅ Erzwingt, dass jede `.ts`-Datei als ein isoliertes Modul behandelt wird.
         *- Erforderlich für den ESBuild- und Babel-Transpiler.
         */
        "isolatedModules": true,

        /*
         *✅ Eingebundene Standardbibliotheken.
         * AKTUELL VERWENDET: ES2023, DOM, DOM.Iterable
         */
        "lib": [
            "ES2023",
            "DOM",
            "DOM.Iterable"
        ],

        /*
         * ═══════════════════════════════════════════════════════════════
         * 📦 MODULE SYSTEM CONFIGURATION
         * ═══════════════════════════════════════════════════════════════
         * ENTERPRISE WAHL: "ESNext"
         */
        "module": "ESNext",

        /*
         * ═══════════════════════════════════════════════════════════════
         * 🔍 MODULE RESOLUTION STRATEGY
         * ═══════════════════════════════════════════════════════════════
         * ENTERPRISE WAHL: "bundler"
         */
        "moduleResolution": "bundler",

        /*
         *✅ Deaktiviert die Ausgabe von `.js`-Dateien.
         *- Wichtig für TypeScript-only-Projekte oder wenn eine separate Build-Pipeline existiert.
         */
        "noEmit": true,

        /*
         *✅ Fehler werfen, wenn `switch`-Statements Fälle ohne `break` haben.
         *- Verhindert unerwartetes Verhalten.
         */
        "noFallthroughCasesInSwitch": true,

        /*
         *✅ Erzwingt explizite Typdeklarationen (kein `any` erlaubt).
         *- Reduziert unerwartetes Verhalten durch dynamische Typen.
         */
        "noImplicitAny": true,

        /*
         *✅ Fehler werfen, wenn eine Funktion nicht explizit `return` hat.
         *- Verhindert Bugs durch unerwartete `undefined`-Rückgaben.
         */
        "noImplicitReturns": true,

        /*
         *✅ Fehler werfen, wenn unbenutzte Variablen vorhanden sind.
         *- Hilft, toten Code zu vermeiden.
         */
        "noUnusedLocals": true,

        /*
         *✅ Fehler werfen, wenn unbenutzte Funktionsparameter vorhanden sind.
         *- Hilft, unnötigen Code zu reduzieren.
         */
        "noUnusedParameters": true,

        /*
         *✅ Definiert TypeScript-Module mit Aliassen.
         */
        "paths": {
            "@/*": ["src/*"],
            "@test/*": ["test/*"]
        },

        /*
         *✅ Erlaubt das Importieren von `.json`-Dateien.
         */
        "resolveJsonModule": true,

        /*
         *✅ Überspringt Typprüfung für Bibliotheken.
         *- Erhöht die Kompiliergeschwindigkeit.
         *- Sollte **deaktiviert** werden, wenn du unsichere Abhängigkeiten prüfst.
         */
        "skipLibCheck": false,

        /*
         *✅ Deaktiviert Source Maps.
         *- Falls Debugging nötig ist, setze auf `true`.
         */
        "sourceMap": true,

        /*
         *✅ Aktiviert den "Strict Mode" für TypeScript.
         */
        "strict": true,

        /*
         *✅ Erzwingt explizite Typdeklarationen (kein `any` erlaubt).
         */
        "strictNullChecks": true,

        /*
         *✅ ECMAScript-Zielversion auf ES2023 gesetzt.
         */
        "target": "ES2023",

        /*
         *✅ Klasseneigenschaften mit `Object.defineProperty` setzen.
         */
        "useDefineForClassFields": true
    },
    "exclude": [
        "node_modules",
        "out"
    ]
}
```

#### `tsconfig.node.json`
```jsonc

{
    /*
     *✅ Verweist auf das TypeScript-Config-Schema.
     */
    "$schema": "https://json.schemastore.org/tsconfig",

    "compilerOptions": {
        /*
         *✅ Erlaubt die Verarbeitung von JavaScript-Dateien.
         */
        "allowJs": true,

        /*
         *✅ Definiert die Basis-URL für die Pfad-Aliase.
         */
        "baseUrl": ".",

        /*
         *✅ Aktiviert "Composite Mode".
         *- Notwendig für "Project References".
         */
        "composite": true,

        "paths": {
            "@/*": ["src/*"],
            "@test/*": ["test/*"]
        },

        /*
         *✅ Definiert globale Typen für das Projekt.
         */
        "types": ["node"]
    },

    /*
     * Extend the base TypeScript configuration
     */
    "extends": "./tsconfig.base.json",

    /*
     *✅ Welche Dateien TypeScript verarbeiten soll.
     */
    "include": [
        "src/**/*",
        "global.d.ts"
    ]
}
```

#### `tsconfig.json`
```jsonc
{
    "files": [],
    "references": [
        {
            "path": "./tsconfig.node.json"
        }
    ]
}
```

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


Was heißt Matrix?

<details><summary>Click to expand..</summary>

In **GitHub Actions** ist eine **Matrix** ein Bauplan für parallele Test-/Build-Varianten. Statt dass du denselben Job manuell mehrmals definierst (z. B. für Node 20 und Node 22, oder für Ubuntu, macOS und Windows), schreibst du nur **eine Job-Definition** und legst eine Matrix fest. GitHub generiert daraus automatisch mehrere unabhängige Job-Runs.

Beispiel aus deinem CI-Snippet:

```yaml
strategy:
  fail-fast: false
  matrix:
    os: [ubuntu-latest]
    node: [20, 22]
```

➡️ Das heißt:

* `os` = Betriebssystem (hier nur Ubuntu, man könnte auch `windows-latest`, `macos-latest` ergänzen).
* `node` = Node.js-Versionen.

Die Matrix multipliziert diese Dimensionen:

* Ubuntu + Node 20
* Ubuntu + Node 22

GitHub startet für jede Kombination einen eigenen Runner. Jeder Job installiert genau die Version, die du in `${{ matrix.node }}` angibst. Das erlaubt dir, früh zu sehen, ob dein Paket auf allen Plattformen und Node-Versionen sauber baut, lintet, testet, etc.

---

### Mehr zu GitHub Actions (im GitHub-Kontext):

* **Workflows**: YAML-Dateien unter `.github/workflows/`. Jede Datei beschreibt, wann (trigger) und wie (jobs) dein CI/CD läuft.
* **Events**: `on: push`, `on: pull_request`, `on: schedule`, usw. – sie lösen einen Workflow aus.
* **Jobs**: Logische Blöcke, laufen parallel (außer man verknüpft sie mit `needs`).
* **Steps**: Befehle innerhalb eines Jobs. Steps können entweder eine Action nutzen (`uses:`) oder ein Shell-Kommando (`run:`).
* **Actions**: Wiederverwendbare Module, wie `actions/checkout` (holt Repo-Code) oder `actions/setup-node` (installiert Node).
* **Permissions**: Feinsteuerung, welche Rechte der Workflow gegenüber dem Repo hat. Best Practice: minimal halten.
* **Caching**: Spart Zeit, indem Abhängigkeiten (`pnpm`, `npm`, `yarn`) nicht jedes Mal frisch gezogen werden.

Im Enterprise-Blueprint oben siehst du:

* **Matrix (OS × Node)** = Multi-Platform/Version Tests.
* **Concurrency** = Stellt sicher, dass nicht mehrere Builds derselben Branch parallel laufen.
* **Minimal Permissions** = Sicherheit (z. B. nur `contents: read`).
* **OIDC-Integration** = Signierte Publishes zu npm ohne statisches Token.

Kurz: **Matrix = Multiplizieren von Testdimensionen**.
GitHub Actions = deine vollautomatische CI/CD-Pipeline direkt im GitHub-Repo, steuerbar über YAML.

👉 Soll ich dir ein konkretes Beispiel bauen, wie die gleiche Matrix für **Node × OS** aussieht (also Ubuntu, macOS, Windows × Node 18, 20, 22), damit du den Effekt live siehst?

</details>

---



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





Was ist OIDC

<details><summary>Click to expand..</summary>

**OIDC to npm** bedeutet, dass dein GitHub-Workflow **OIDC (OpenID Connect) verwendet, um sich bei npm zu authentifizieren**, anstatt ein statisches `NPM_TOKEN` im Repository zu speichern.

Hier die Details:

---

### 1️⃣ OIDC = OpenID Connect

* Standard für **identity federation**: GitHub kann gegenüber externen Services beweisen, dass ein Workflow vertrauenswürdig ist.
* GitHub erstellt dafür **ein JWT (JSON Web Token)** für den Workflow-Run.
* Dieses Token kann **dynamisch** an npm weitergegeben werden.

---

### 2️⃣ Problem ohne OIDC

* Früher musste man **ein npm-Token als Secret** im GitHub-Repo speichern: `NPM_TOKEN`.
* Risiko: Token könnte geleakt werden, wenn jemand Zugriff auf das Repo hat.

---

### 3️⃣ Mit OIDC

* Workflow kann `id-token: write` in den Berechtigungen setzen:

```yaml
permissions:
  contents: write
  id-token: write
```

* Dann kann die Action (`npm publish`) **mit JWT statt Token** authentifizieren.
* npm prüft: „Dieses Token wurde von GitHub erzeugt, Workflow vertrauenswürdig → Veröffentlichung erlaubt.“

---

### 4️⃣ Vorteile

* **Kein statisches Secret mehr nötig** → sicherer.
* Tokens **laufen nach wenigen Minuten ab** → reduziert Risiko von Missbrauch.
* Passt perfekt zu automatisierten CI/CD-Releases über **Changesets + GitHub Actions**.

---

💡 **Kurz gesagt:**

> OIDC to npm = GitHub sagt npm: „Vertraue diesem Workflow, er darf veröffentlichen“ – ohne dass du ein dauerhaftes Token speichern musst.

---

Wenn du willst, kann ich auch zeigen, wie ein **Release-Job mit OIDC für npm** in GitHub Actions **komplett aussieht**, inkl. JWT-Token-Flow. Willst du das?


</details>




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


Was ist CodeQL?

<details><summary>Click to expand..</summary>

**CodeQL** ist ein **sicherheitsfokussiertes Analyse-Tool von GitHub**, das deinen Code auf **Schwachstellen, Bugs und Sicherheitsrisiken** untersucht. Es ist ein bisschen wie eine **SQL-Datenbank für Code**: man kann Abfragen schreiben, um bestimmte Muster im Code zu finden.

---

### 1️⃣ Grundprinzip

* Dein Code wird in **eine Datenbank transformiert**.
* Diese Datenbank kann dann mit **CodeQL-Abfragen** durchsucht werden.
* Beispiel: Prüfe, ob irgendwo in Node.js ein unsicheres `eval()` auf Benutzereingaben angewendet wird.

---

### 2️⃣ Einsatz in GitHub Actions

* Workflow läuft z. B. auf `pull_request` oder `push`.
* Schritte in deinem Beispiel:

```yaml
- uses: github/codeql-action/init@v3
  with: { languages: javascript-typescript }

- uses: github/codeql-action/analyze@v3
```

* **Init:** erstellt CodeQL-Datenbank für die Sprache(n).
* **Analyze:** führt die Abfragen aus und erstellt einen Report (SARIF-Datei).

---

### 3️⃣ Vorteile

* Findet **Sicherheitslücken früh im CI**, bevor Code deployed wird.
* Unterstützt viele Sprachen (JS/TS, Python, Java, C#, C/C++...).
* Automatisiert Sicherheitschecks und lässt sich in **Pull-Request-Reviews** integrieren.

---

💡 **Kurz gesagt:**

> CodeQL = GitHubs „SQL für Code“, um systematisch Sicherheitslücken, Bugs und riskanten Code zu erkennen.



</details>

---

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


Was ist OpenSSF Scorecard?

<details><summary>Click to expand..</summary>

Die **OpenSSF Scorecard** ist ein **automatisches Sicherheits-Audit-Tool für Open-Source-Projekte**, entwickelt von der **Open Source Security Foundation (OpenSSF)**.

---

### 1️⃣ Zweck

* Bewertet **Best Practices** für Sicherheit in einem Repository.
* Gibt **Punkte** oder ein Rating für jede Kategorie.
* Ziel: Risiken früh erkennen und OSS-Projekte sicherer machen.

---

### 2️⃣ Bewertete Kategorien (Beispiele)

* **Code-Review**: Werden Pull Requests geprüft?
* **Vulnerability Management**: Gibt es Dependabot/Remediate-Prozesse?
* **Branch Protection**: Ist der `main`-Branch geschützt?
* **Signed Commits**: Werden Commits signiert?
* **CI-Security**: Sind CI-Workflows abgesichert?
* **Licensing**: Klare Lizenz vorhanden?

---

### 3️⃣ Einsatz in GitHub Actions

Beispiel aus deinem Blueprint:

```yaml
- uses: ossf/scorecard-action@v2.3.3
  with: { results_file: results.sarif }
- uses: github/codeql-action/upload-sarif@v3
  with: { sarif_file: results.sarif }
```

* Läuft automatisiert auf PRs oder nach Zeitplan (`cron`).
* Generiert **SARIF-Report**, der in GitHub Security Tab angezeigt wird.

---

### 4️⃣ Vorteile

* Automatisiertes Security-Checkup für Repositories.
* Zeigt **potenzielle Sicherheitslücken oder fehlende Praktiken** auf.
* Unterstützt OSS- und Enterprise-Projekte gleichermaßen.
* Ergänzt Tools wie CodeQL, Dependabot oder Renovate.

---

💡 **Kurz gesagt:**

> OpenSSF Scorecard = automatisches „Security Health Check“ für ein GitHub-Repo, das zeigt, wie gut Sicherheitsbest Practices eingehalten werden.


</details>


<br><br>

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

<br>

Was heißt das?

<details><summary>Click to expand..</summary>



### 1️⃣ `"bin"` in `package.json`

```json
"bin": { "pkg-cli": "./dist/cli.js" }
```

* **`bin`** definiert, welche Datei als **ausführbares Kommando** verfügbar sein soll, wenn jemand dein Paket global installiert (`npm install -g pkg`).
* `pkg-cli` = Name des CLI-Kommandos.
* `./dist/cli.js` = die JS-Datei, die ausgeführt wird, wenn man `pkg-cli` im Terminal tippt.

➡️ Ergebnis: Nach Installation kannst du einfach `pkg-cli` in der Shell ausführen.

---

### 2️⃣ Shebang

```ts
#!/usr/bin/env node
```

* Zeile ganz oben in `cli.js`.
* Sagt dem Betriebssystem:

  > „Führe diese Datei mit Node.js aus.“
* Standard für **Node‑CLI-Programme** auf Linux/macOS.

---

### 3️⃣ ESM Loader

```ts
import('node:process'); // keep ESM; no top-level await if Node 20 target is OK
```

* Dynamischer Import (`import()` statt `require`) → **ESM-kompatibel**.
* Node 20 erlaubt dynamische Imports in CLI, ohne Top-Level-Await zu benutzen.
* Stellt sicher, dass dein CLI **ESM-Modul** ist, auch wenn es global installiert wird.

---

### 🔹 Zusammenfassung

1. `"bin"` → definiert CLI-Kommando.
2. `#!/usr/bin/env node` → macht die JS-Datei ausführbar auf Unix-Systemen.
3. Dynamischer Import → kompatibel mit **ESM + Node 20+**, ohne dass CLI beim Start blockiert wird.

</details>



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