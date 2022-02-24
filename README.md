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
