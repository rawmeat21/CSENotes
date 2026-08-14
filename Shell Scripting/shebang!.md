https://tldp.org/LDP/abs/html/sha-bang.html

```
#!/bin/sh
#!/bin/bash
#!/usr/bin/perl
#!/usr/bin/tcl
#!/bin/sed -f
#!/bin/awk -f
```

Using `#!/bin/sh`, the default Bourne shell in most commercial variants of UNIX, makes the script [portable](https://tldp.org/LDP/abs/html/portabilityissues.html) to non-Linux machines, though you [sacrifice Bash-specific features](https://tldp.org/LDP/abs/html/gotchas.html#BINSH). The script will, however, conform to the POSIX [[5]](https://tldp.org/LDP/abs/html/sha-bang.html#FTN.AEN256) **sh** standard.

`#!/bin/sh` invokes the default shell interpreter, which defaults to /bin/bash on a Linux machine.



Why not simply invoke the script with `scriptname`? If the directory you are in ([$PWD](https://tldp.org/LDP/abs/html/internalvariables.html#PWDREF)) is where scriptname is located, why doesn't this work? This fails because, for security reasons, the current directory (./) is not by default included in a user's [$PATH](https://tldp.org/LDP/abs/html/internalvariables.html#PATHREF). It is therefore necessary to explicitly invoke the script in the current directory with a **./scriptname**.

