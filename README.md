While inside the repo root, deploy to remote server with rsync:

```
$ hugo && rsync -e 'ssh -i <my private key path>' -avz --delete public/ <user-name>@<remote server ip>:<destination path>
```