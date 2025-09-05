## ESLint-Plugin-Enterprise: Cheat‑Sheet (lokale Entwicklung mit Link)

### TSUP Build (im Plugin)
- **Ziel**: ESM‑Artefakte für Node 20+ mit DTS
- **Kern-Settings**:
  - **clean**: true
  - **entry**: ['src/index.ts']
  - **format**: ['esm']  ← erzeugt `dist/index.js`
  - **dts**: { resolve: true, compilerOptions: { composite: false } }  ← erzeugt `dist/index.d.ts`
  - **sourcemap**: true
  - **treeshake**: true
  - **tsconfig**: 'tsconfig.json'

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

### package.json (im Plugin)
- **ESM** + **Exports** + **Types** zeigen auf die gebauten Artefakte in `dist/`
- `main` ist optional, kann auf dasselbe ESM‑Entry zeigen

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

### package.json (im Consumer)
- **Lokaler Link** auf das Plugin‑Repo

```json
{
  "devDependencies": {
    "eslint-plugin-enterprise": "link:C:/Projects/programming-languages/typescript/linting/eslint-plugin-enterprise"
  }
}
```

### Verwendung im Consumer (`eslint.config.ts`)
- **Import** und **Nutzung** des Flat Config‑Presets

```ts
import { enterprisePlugin } from 'eslint-plugin-enterprise'

// …
enterprisePlugin.configs.all,
```

### Befehle (Dev‑Loop)
- **Plugin bauen/watchen**:
```bash
pnpm -C "C:\Projects\programming-languages\typescript\linting\eslint-plugin-enterprise" dev
# oder einmalig:
pnpm -C "C:\Projects\programming-languages\typescript\linting\eslint-plugin-enterprise" build
```
- Wenn wir den **Dev-Modus** benutzen, dann aktualisieren sich die **Dateien** automatisch im **Consumer-Projekt** bei **Node-Module**.

- **Consumer aktualisieren**:
```bash
pnpm -C "C:\git\privadent\privadent-synchronizer" install --force
```
- Danach ggf. **TS‑Server/IDE neu starten**.

### Wichtige Stolpersteine
- **Exports ↔ Build**: `"exports"`/`"types"` müssen exakt zu den gebauten Dateien passen (`dist/index.js`, `dist/index.d.ts`).
- **CJS vs ESM**: Wenn `format` nicht `esm` ist, entstehen `.cjs`/`.d.cts` → dann `"exports"`/`"types"` anpassen.
- **Engines**: Achte auf kompatible Node‑Versionen (Plugin `"engines"` vs Consumer‑Node).
