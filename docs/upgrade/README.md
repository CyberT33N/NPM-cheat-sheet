# Upgrade
- Upgrade package

```bash
npm upgrade request

# or
npm install mongoose-model-manager@latest
```


Um in einem **MPM-Projekt** (ich nehme an, du meinst "Monorepo mit `pnpm`" oder ähnliches – falls nicht, bitte kurz klären) in der `package.json` **alle Dependencies inklusive Major-Updates und Breaking Changes** zu aktualisieren, gibt es mehrere Wege. Der radikale, aber effektive Weg:

---

### ✅ **1. Automatisch mit `ncu` (npm-check-updates)**

#### 🔧 Installation (falls nicht global installiert):

```bash
npm install -g npm-check-updates
```

#### 🚀 Alle Dependencies inkl. Major-Versionen aktualisieren:

```bash
ncu -u
```

Das aktualisiert deine `package.json` mit den **neuesten** Versionen (auch Breaking Changes).

#### 📦 Danach: Installieren

```bash
npm install
```

---

### 🧨 Optional: Auch DevDependencies mitnehmen

```bash
ncu -u --dep dev
```

Oder **beides zusammen**:

```bash
ncu -u --target latest
npm install
```

---

### 💡 Für ein **MPM/Monorepo mit Workspaces** (`pnpm`, `yarn`, `npm`):

Falls du ein Monorepo mit mehreren `package.json`-Dateien hast (z. B. mit `pnpm workspaces`), kannst du in der **Root**-Struktur ausführen:

```bash
npx npm-check-updates -u -w
```

Oder per Script über alle `packages/*` iterieren:

```bash
find packages -name package.json -execdir ncu -u \;
```

Dann global installieren:

```bash
pnpm install
```

---

### 🔥 Bonus: Komplett radikal – alles auf absolut neu und dann Fehler fixen

```bash
ncu -u --target latest
rm -rf node_modules package-lock.json
npm install
```

Oder für `pnpm`:

```bash
ncu -u --target latest
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

---

### 🧠 Warum nicht `npm update`?

Weil `npm update` **keine Major-Versionen** updated – es bleibt innerhalb des semver-Rahmens (`^` oder `~`).

---

Wenn du mutig bist, kannst du mit `--doctor` von `ncu` testen, ob dein Projekt noch funktioniert.

---

### ⚠️ Danach: Tests fahren. Breaking heißt oft wirklich **breaking**.

---

Wenn du willst, kann ich dir auch ein Skript machen, das deine ganzen Monorepo-Packages durchballert. Sag nur Bescheid 😎
