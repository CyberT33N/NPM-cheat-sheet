Here’s the full 1:1 translation into English:

---

## eslint-plugin-test: Cheat-Sheet (local development with link)

### TSUP Build (in the plugin)

* **Goal**: ESM artifacts for Node 20+ with DTS
* **Core settings**:

  * **clean**: true
  * **entry**: \['src/index.ts']
  * **format**: \['esm']  ← produces `dist/index.js`
  * **dts**: { resolve: true, compilerOptions: { composite: false } }  ← produces `dist/index.d.ts`
  * **sourcemap**: true
  * **treeshake**: true
  * **tsconfig**: 'tsconfig.json'

```ts
import { defineConfig } from 'tsup'

export default defineConfig({
  clean: true,
  entry: ['src/index.ts'],
  format: ['esm'],
  dts: { resolve: true, compilerOptions: { composite: false } },
  sourcemap: true,
  treeshake: true,
  tsconfig: 'tsconfig.json'
})
```

### package.json (in the plugin)

* **ESM** + **Exports** + **Types** point to the built artifacts in `dist/`
* `main` is optional, can point to the same ESM entry

```json
{
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "./package.json": "./package.json"
  },
  "files": [
    "dist/",
    "LICENSE",
    "README.md",
    "!dist/**/*.map",
    "!**/*.test.*",
    "!**/*.spec.*",
    "!**/__tests__/"
  ],
  "scripts": {
    "build": "tsup",
    "dev": "tsup --watch --config tsup.config.ts",
    "preinstall": "npx only-allow pnpm",
    "prepare": "husky",
    "pack:dry": "npm pack --dry-run"
  }
}
```

### package.json (in the consumer)

* **Local link** to the plugin repo

```json
{
  "devDependencies": {
    "eslint-plugin-test": "link:C:/Projects/programming-languages/typescript/linting/eslint-plugin-test"
  }
}
```

### Usage in consumer (`eslint.config.ts`)

* **Import** and **use** the Flat Config preset

```ts
import { enterprisePlugin } from 'eslint-plugin-test'

// …
enterprisePlugin.configs.all,
```

### Commands (dev loop)

* **Build/watch the plugin**:

```bash
pnpm -C "C:\Projects\programming-languages\typescript\linting\eslint-plugin-test" dev
# or once:
pnpm -C "C:\Projects\programming-languages\typescript\linting\eslint-plugin-test" build
```

* When using **dev mode**, the **files** in the **consumer project** under **node\_modules** update automatically.

* **Update consumer**:

```bash
pnpm -C "C:\git\test\test-synchronizer" install --force
```

* Afterwards, restart **TS server/IDE** if necessary.

### Important pitfalls

* **Exports ↔ Build**: `"exports"`/`"types"` must match the built files exactly (`dist/index.js`, `dist/index.d.ts`).
* **CJS vs ESM**: If `format` is not `esm`, then `.cjs`/`.d.cts` are created → update `"exports"`/`"types"` accordingly.
* **Engines**: Make sure Node versions are compatible (plugin `"engines"` vs consumer Node).
