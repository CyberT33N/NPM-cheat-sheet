# NPM Cheat Sheet
NPM Cheat Sheet with the most needed stuff..



# Login
```bash
npm login
```



















<br><br>
__________________________________________
__________________________________________
<br><br>

# Install 

<details><summary>Click to expand..</summary>
 
## Install package
```bash
npm i request
```

<br><br>

## Install package to devDependencies
```bash
npm i --save-dev sinon 
```




<br><br>

## Install package with security audit check
```bash
npm i --no-audit sinon 
```


</details>
























































<br><br>
<br><br>



<br><br>
 _____________________________________________________
 _____________________________________________________
<br><br>


# package.json

<details><summary>Click to expand..</summary>
 
<br><br>

## General Informations
- https://docs.npmjs.com/creating-a-package-json-file


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



















































<br><br>
__________________________________________
__________________________________________
<br><br>

# Workspaces 

<br><br>

## List all workspaces
```bash
npm ls -ws

npm ls --production --depth 1 -json | jq -r '.dependencies[].resolved' | sed 's/file:\.\.\///
```































<br><br>
__________________________________________
__________________________________________
<br><br>

# Upgrade
- Upgrade package

```bash
npm upgrade request
```













































































<br><br>
__________________________________________
__________________________________________
<br><br>

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
























<br><br>
__________________________________________
__________________________________________
<br><br>

# scripts (https://docs.npmjs.com/cli/v7/using-npm/scripts)

<details><summary>Click to expand..</summary>


<br><br>

## run (https://docs.npmjs.com/cli/v7/commands/npm-run-script)
- This runs an arbitrary command from a package's "scripts" object. If no "command" is provided, it will list the available scripts.

<br>

run[-script] is used by the test, start, restart, and stop commands, but can be called directly, as well. When the scripts in the package are printed out, they're separated into lifecycle (test, start, restart) and directly-run scripts.

<br>

Any positional arguments are passed to the specified script. Use -- to pass --prefixed flags and options which would otherwise be parsed by npm.

<br>

For example:
```bash
// package.json
{
  "scripts": {
    "test": "mocha ./test/integration/**/*.test.js --exit"
  }
}

npm run test
```





<br><br>

## postinstall (https://docs.npmjs.com/cli/v7/commands/npm-run-script)
- This will be automatically executed when you install your project with **npm i**
```bash
// package.json
{
  "scripts" : {
    "postinstall" : "bash crazyscript.sh",
  }
}

npm i
```


</details>






























<br><br>
__________________________________________
__________________________________________
<br><br>

# ls
- find folder path of node package inside of your node_modules folder
```bash
npm ls packagename

// you can search aswell global
npm ls -g packagename
```






































<br><br>
__________________________________________
__________________________________________
<br><br>

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

















<br><br>
<br><br>
<br><br>
<br><br>

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



<details>




































<br><br>
__________________________________________
__________________________________________
<br><br>

# Errors and Fixes

<br><br>

## No matching version found for ***
```bash
npm cache clean --force
```



 No matching version found for
