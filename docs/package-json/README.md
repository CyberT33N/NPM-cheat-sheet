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
