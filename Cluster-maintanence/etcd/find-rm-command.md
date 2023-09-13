
# Find command

```sh
- find /opt/etcd-backup -type f -mtime +30 -name '*.db' -exec rm -fr {} \;
```

- The **“/opt/etcd-backup”** argument specifies the current directory, 
- The **“-type f”** argument specifies that we want to search for files (not directories), 
- The **“-mtime +30”** argument specifies that we want to search for files that are older than 30 days,
- The **"-name '*.db'"** argument specifies that file name,
- The **"-exec"** argument specifies that the comand we want to execute.