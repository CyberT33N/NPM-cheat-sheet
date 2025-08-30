# Workspaces 

<br><br>

## List all workspaces
```bash
npm ls -ws

npm ls --production --depth 1 -json | jq -r '.dependencies[].resolved' | sed 's/file:\.\.\///
```
