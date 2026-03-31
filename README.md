# WSL-notes
Windows Subsystem for Linux - cheatsheet


## Safe mounts (No auto mounting of host drives, no auto path, read-only mounts for VSCode and Cursor only:


Modify /etc/wsl.conf (disable automount and auto append paths from host):

```
[automount]
enabled=false
mountFsTab = true

[interop]
appendWindowsPath = false
```

Then create targeted mount dirs, fstab and wrappers for VSCode and Cursor:
```
if [ "$EUID" -ne 0 ]; then
USER_HOME="/mnt/c/Users/$USER"

VSCODE_WIN="C:\\Users\\$USER\\AppData\\Local\\Programs\\Microsoft\040VS\040Code\\"
VSCODE_EXT_WIN="C:\\Users\\$USER\\.vscode\\extensions\\"
VSCODE_LINUX="$USER_HOME/AppData/Local/Programs/Microsoft VS Code"
VSCODE_LINUX_FSTAB="$USER_HOME/AppData/Local/Programs/Microsoft\040VS\040Code"
VSCODE_EXT_LINUX="$USER_HOME/.vscode/extensions"
VSCODE_EXT_LINUX_FSTAB="$VSCODE_EXT_LINUX"

CURSOR_WIN="C:\\Users\\$USER\\AppData\\Local\\Programs\\cursor\\"
CURSOR_EXT_WIN="C:\\Users\\$USER\\.cursor\\extensions\\"
CURSOR_LINUX="$USER_HOME/AppData/Local/Programs/cursor"
CURSOR_LINUX_FSTAB="$CURSOR_LINUX"
CURSOR_EXT_LINUX="$USER_HOME/.cursor/extensions"
CURSOR_EXT_LINUX_FSTAB="$CURSOR_EXT_LINUX"

sudo mkdir -p "$VSCODE_LINUX"
sudo mkdir -p "$VSCODE_EXT_LINUX"
sudo mkdir -p "$CURSOR_LINUX"
sudo mkdir -p "$CURSOR_EXT_LINUX"

FSTAB_LINES=$(cat <<EOF
$VSCODE_WIN  $VSCODE_LINUX_FSTAB  drvfs  ro  0  0
$VSCODE_EXT_WIN  $VSCODE_EXT_LINUX_FSTAB  drvfs  ro  0  0
$CURSOR_WIN  $CURSOR_LINUX_FSTAB  drvfs  ro  0  0
$CURSOR_EXT_WIN  $CURSOR_EXT_LINUX_FSTAB  drvfs  ro  0  0
EOF
)

echo "$FSTAB_LINES" | while read -r line; do
    if ! grep -Fq "$line" /etc/fstab; then
        echo "$line" | sudo tee -a /etc/fstab >/dev/null
    fi
done

if [ ! -f /usr/local/bin/code ]; then
    sudo tee /usr/local/bin/code >/dev/null <<EOF
#!/bin/bash
"$VSCODE_LINUX/bin/code" "\$@"
EOF
    sudo chmod +x /usr/local/bin/code
fi

if [ ! -f /usr/local/bin/cursor ]; then
    sudo tee /usr/local/bin/cursor >/dev/null <<EOF
#!/bin/bash
"$CURSOR_LINUX/resources/app/bin/cursor" "\$@"
EOF
    sudo chmod +x /usr/local/bin/cursor
fi

else
    echo "This script must be run as a normal user, not as root."
fi
sudo systemctl daemon-reload
```

Shutdown WSL with (care, this will stop ALL your WSLs):

    wsl --shutdown 

Check mounts:
```
mount | grep C:
C:\Users\user\AppData\Local\Programs\Microsoft VS Code\ on /mnt/c/Users/user/AppData/Local/Programs/Microsoft VS Code type 9p (ro,relatime,aname=drvfs;path=C:\Users\user\AppData\Local\Programs\Microsoft)                                                                            
C:\Users\user\.vscode\extensions\ on /mnt/c/Users/user/.vscode/extensions type 9p (ro,relatime,aname=drvfs;path=C:\Users\user\.vscode\extensions\;symlinkroot=/mnt/,cache=5,access=client,msize=65536,trans=fd,rfd=3,wfd=3)                                                            
C:\Users\user\AppData\Local\Programs\cursor\ on /mnt/c/Users/user/AppData/Local/Programs/cursor type 9p (ro,relatime,aname=drvfs;path=C:\Users\user\AppData\Local\Programs\cursor\;symlinkroot=/mnt/,cache=5,access=client,msize=65536,trans=fd,rfd=3,wfd=3)                           
C:\Users\user\.cursor\extensions\ on /mnt/c/Users/user/.cursor/extensions type 9p (ro,relatime,aname=drvfs;path=C:\Users\user\.cursor\extensions\;symlinkroot=/mnt/,cache=5,access=client,msize=65536,trans=fd,rfd=3,wfd=3)                                               
```

Check /etc/fstab:
```
# UNCONFIGURED FSTAB FOR BASE SYSTEM

C:\Users\user\AppData\Local\Programs\Microsoft\040VS\040Code\  /mnt/c/Users/user/AppData/Local/Programs/Microsoft\040VS\040Code  drvfs  ro  0  0
C:\Users\user\.vscode\extensions\  /mnt/c/Users/user/.vscode/extensions  drvfs  ro  0  0
C:\Users\user\AppData\Local\Programs\cursor\  /mnt/c/Users/user/AppData/Local/Programs/cursor  drvfs  ro  0  0
C:\Users\user\.cursor\extensions\  /mnt/c/Users/user/.cursor/extensions  drvfs  ro  0  0
```

Check /usr/local/bin/code:
```
#!/bin/bash
"/mnt/c/Users/user/AppData/Local/Programs/Microsoft VS Code/bin/code" "$@"
```

Check /usr/local/bin/cursor:
```
#!/bin/bash
"/mnt/c/Users/user/AppData/Local/Programs/cursor/resources/app/bin/cursor" "$@"
```

Test VSCode:

    mkdir testcode ; cd testcode ; code .

Test Cursor:

    mkdir testcursor ; cd testcursor ; cursor .
