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
