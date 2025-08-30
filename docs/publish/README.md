# 📦 Publish

<details><summary>Click to expand..</summary>

## 🤖 Automating Releases and Publishing Packages to NPM Using GitHub Actions
- Read more: [Automate Releases and Publish Packages to NPM](https://nanthakumaran.medium.com/how-to-automate-releases-and-publish-packages-to-npm-using-github-actions-910d5128c0fa)

### 1. Generate Required Tokens
To automate releases in your github repo and publish packages to npm registry, you will need two tokens:
- **PA_TOKEN**: Personal Access Token from GitHub
- **NPM_TOKEN**: NPM Token for automation

### 2. Create Your PA_TOKEN
Visit [GitHub Token Settings](https://github.com/settings/tokens/new) to generate a new Personal Access Token. 
- **Select the following scopes**:
  - `workflow` 
  - `write:packages`

### 3. Create Your NPM_TOKEN
1. Navigate to **Settings > Access Tokens > Generate new token > Classic token**.
2. Ensure you select **Automation**.

### 4. Add NPM Token to GitHub Secrets
1. Open the repository's **Settings**.
2. Under **Security > Secrets > Actions**, click **New Repository Secret** to add your tokens.
   - In the **Name** field, enter `PA_TOKEN` and in the **Value** field, paste your token.
   - Repeat for `NPM_TOKEN`.
  
### 5. Workflow permissions
- Go in your project to Settings > Actions (https://github.com/CyberT33N/ModelManager/settings/actions)
  - Scroll down to `Workflow permissions` and check:
    - `Read and write permissions`
    - `Allow GitHub Actions to create and approve pull requests `

### 6. Set Up Workflow Files in Your Project
Create the following files in your project structure:

#### .github/workflows/release.yml
```yaml
name: Releases
on:
  push:
    branches:
      - main

jobs:
  changelog:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Conventional Changelog Action
        id: changelog
        uses: TriPSs/conventional-changelog-action@v3.7.1
        with:
          github-token: ${{ secrets.PA_TOKEN }}
          version-file: './package.json,./package-lock.json'
      - name: Create Release
        uses: actions/create-release@v1
        if: ${{ steps.changelog.outputs.skipped == 'false' }}
        env:
          GITHUB_TOKEN: ${{ secrets.PA_TOKEN }}
        with:
          tag_name: ${{ steps.changelog.outputs.tag }}
          release_name: ${{ steps.changelog.outputs.tag }}
          body: ${{ steps.changelog.outputs.clean_changelog }}
```
- **Note**: This workflow will create a release on the `main` branch.

#### .github/workflows/publish.yml
```yaml
name: Publish to NPM
on:
  release:
    types: [created]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '16'
          registry-url: 'https://registry.npmjs.org'
      - name: Install Dependencies and Build 🔧
        run: npm ci
      - name: Publish Package to NPM 📦
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Summary of the Workflow
1. The first entry point is `release.yml`, which generates a new release based on conventional commits when you commit code.
2. After each valid commit, `release.yml` updates `CHANGELOG.md`, `package.json`, and `package-lock.json`.
3. Once the release is created, `publish.yml` publishes the package to the NPM Registry.

### 📥 Workflow for Committing Changes
Whenever you make changes, follow these steps:

```bash
# Pull the latest created release
git pull

git add .
# Refer to the Conventional Commits documentation: https://www.conventionalcommits.org/en/v1.0.0/
git commit -m "feat: Added payment feature"

# Example for breaking changes
# git commit -m "feat: renamed error interfaces" -m "BREAKING CHANGE: renamed error interfaces"

git push

# If conflicts arise due to automatic versioning in package.json, execute:
git pull --rebase

# Resolve any conflicts
# git add .
# git rebase --continue

git push -f
```

### 📦 Publishing Your Package
To publish the package to NPM, ensure your `package.json` is valid, then run:
```bash
npm init
npm publish
```
- **Important**: Ensure that your `package.json` name is unique and does not already exist in the NPM registry (check [npmjs.com](https://www.npmjs.com/search?q=xxxxx)).


























## TypeScript NPM Package Publishing: A Beginner’s Guide
- https://pauloe-me.medium.com/typescript-npm-package-publishing-a-beginners-guide-40b95908e69c

1. npm init und projekt erstellen
- If exists remove "type": "module" from package.json 

2. Install dependencies
```shell
npm install --save-dev typescript ts-node
npm i tsup glob
```

3. Setup your tsconfig.json , run the following command npx tsc --init , it will create a tsconfig file in the base of your project. Update the outDir field to "dist"

4. Create .gitignore:
```
/node_modules

# Ignore test-related files
/coverage.data
/coverage/

# Build files
/dist
```

5. Create tsup.config.ts:
```javascript
import { defineConfig } from "tsup"
import * as glob from 'glob'

const entries = glob.sync('./src/*.ts')
console.log(entries)

export default defineConfig({
  entry: entries,
  format: ["cjs", "esm"], // Build for commonJS and ESmodules
  dts: true, // Generate declaration file (.d.ts)
  splitting: false,
  sourcemap: true,
  clean: true
})
```

6. Create npm run scripts
```javascript
"scripts": {
    "build": "tsup",
    "test": "echo \"Error: no test specified\" && exit 1"
},
```

7. Add to package.json
```javascript
"main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    "import": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js"
    },
    "require": {
      "types": "./dist/index.d.cts",
      "require": "./dist/index.cjs"
    }
  },
"files": [
    "dist"
 ],
```

8. Create an account on NPM if you don’t already have one.
- https://www.npmjs.com/signup

9. Run `npm login`

10. Run `npm publish`





<br><br>
<br><br>

## Unpublish (Delete package from the registry)
- https://docs.npmjs.com/unpublishing-packages-from-the-registry/
```bash
npm unpublish --force your package
```



</details>
