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

# .npmrc

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
