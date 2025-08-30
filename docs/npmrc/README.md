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
