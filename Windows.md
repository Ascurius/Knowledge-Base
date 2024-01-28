### Managing files

Find all files of certain type in directory and subdirectories and move them to specified target directory

```ps1
Get-ChildItem -Path "Y:\Serien\Lost-in-Space\Staffel-1" -Recurse -File -Filter "*.mkv" | Move-Item -Destination "Y:\Serien\Lost-in-Space\Staffel-1" -Force
```

### Networking

Port forwarding
```cmd
netsh interface portproxy add v4tov4 listenport=SOURCE_PORT listenaddress=SOURCE_IP connectport=TARGET_PORT connectaddress=TARGET_IP
```

Remove the port forwarding

```cmd
netsh interface portproxy delete v4tov4 listenport=SOURCE_PORT listenaddress=SOURCE_IP connectport=TARGET_PORT connectaddress=TARGET_IP
```