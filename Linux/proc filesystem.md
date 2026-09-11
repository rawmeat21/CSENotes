
The Linux versions of `ps` and `top` read their process status information from the
`/proc` directory, **a pseudo-filesystem in which the kernel exposes a variety of interesting information about the system’s state.**

The information is NOT limited to process information.

Note: Because the kernel creates the contents of `/proc` files on the fly (as they are read),
most appear to be empty, 0-byte files when listed with `ls -l`. You’ll have to `cat` or `less`
the contents to see what they actually contain.

![[Pasted image 20260911233227.png]]

**The individual components contained within the cmdline and environ files are
separated by null characters rather than newlines. You can filter their contents
through `tr "\000" "\n"` to make them more readable.


