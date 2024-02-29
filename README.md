# NPM-cheat-sheet
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



























































<br><br>
<br><br>



<br><br>
 _____________________________________________________
 _____________________________________________________
<br><br>


# package.json

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

## Set main file
- Set a custom file which should be executed in the beginning when you run your app. If you do not define any file then first index.js will be choosed. If index.js does not exist then server.js will be choosed.
```bash
{ "main": "src/index.js" }
```

<br><br>

#### execute async function inside of main file
```bash
const main = async () => {
    //..
}

main().then(() => {
    console.log('main() has been called')
})
```


<br><br>

## Create custom license
```javascript
// Then include a file named <filename> at the top level of the package.
{ "license" : "SEE LICENSE IN <filename>" }
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

<br><br>

## Use nodemon with test
```javascript
"scripts": {
  "test": "mocha test.js",
  "test-watch": "nodemon --exec \"npm test\""
}
```
```bash
npm run test-watch
```






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

# Unpublish (Delete package from the registry)
- https://docs.npmjs.com/unpublishing-packages-from-the-registry/
```bash
npm unpublish --force your package
```








































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
