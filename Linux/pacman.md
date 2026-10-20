```bash
pacman -Qdt
```

Gives the orphan list: packages installed as dependencies that no installed package requires anymore. Means that we can try deleting them. 


```bash
pacman -Rsp <pkg_name>
```

This will tell you what will get deleted if you deleted `<pkg_name>`. Why are multiple packages deleted? That's because `<pkg_name>` has them as a dependency, AND no other package has them as a dependency. If you imagine a dependency graph then it means that only `<pkg_name>` has an arrow pointing to them.



```bash
pacman -Rns <pkg_name>
```

This deletes everything in `pacman -Rsp <pkg_name>`.




```bash
pacman -Qi <pkg_name>
```

This gives you more information about `<pkg_name>`.

An example on my system:

```bash
❯ pacman -Qi go    
Name            : go
Version         : 2:1.27.2-1
Description     : Core compiler tools for the Go programming language
Architecture    : x86_64
URL             : https://go.dev/
Licenses        : BSD-3-Clause
Groups          : None
Provides        : go-pie
Depends On      : None
Optional Deps   : None
Required By     : None
Optional For    : None
Conflicts With  : None
Replaces        : go-pie
Installed Size  : 226.64 MiB
Packager        : Lukas Fleischer <lfleischer@archlinux.org>
Build Date      : Fri 09 Oct 2026 05:48:18 AM IST
Install Date    : Tue 20 Oct 2026 02:38:49 PM IST
Install Reason  : Explicitly installed
Install Script  : No
Validated By    : Signature
```



```bash
grep "<pkg-name>" /var/log/pacman.log
```

shows you logs about the package.



```bash
cat /etc/pacman.conf
```
See which repositories are being searched by pacman.

Some may specify an include file, you can check it out just by `cat`ing it.



```bash
pactree <pkg_name>
```
Will show you the dependency tree of `<pkg_name>`.

