The shebang (`#!`) ==tells the operating system which program to use to run your script==

**Direct Execution:** Allows you to run a text file like `./script.py` instead of typing `python3 script.py`.

**Ignores Extensions:** Unix systems do not rely on file extensions like `.py` or `.sh` to figure out file types; they read the shebang instead.

Common Examples

- `#!/bin/bash` runs a script using the Bash shell at a fixed path.

- `#!/usr/bin/env python3` finds Python 3 dynamically using your system path, which is safer for cross-platform use.


Dont use it?

**Shell fallback:** If you run a script without a shebang in a terminal, the terminal will usually try to run it using your current shell (like Bash or Zsh). If it is a Python or Node.js script, it will fail immediately with syntax errors.

You also need execute permissions. Otherwise do:

```
bash script.sh
```

![Pasted image 20260807104953](../assets/Pasted%20image%2020260807104953.png)

![Pasted image 20260807105156](../assets/Pasted%20image%2020260807105156.png)

 ![Pasted image 20260807105309](../assets/Pasted%20image%2020260807105309.png)


![Pasted image 20260807105418](../assets/Pasted%20image%2020260807105418.png)

![Pasted image 20260807110013](../assets/Pasted%20image%2020260807110013.png)

**hostname is a command, this is how you get the output of commands

![Pasted image 20260807110111](../assets/Pasted%20image%2020260807110111.png)


![Pasted image 20260807110257](../assets/Pasted%20image%2020260807110257.png)

Note: space separated values!

![Pasted image 20260807110614](../assets/Pasted%20image%2020260807110614.png)

`myArray[*]` gets all the values in the array.

![Pasted image 20260807110927](../assets/Pasted%20image%2020260807110927.png)

To get subarray, the syntax is `$arr[*]:start:number of items you want`

![Pasted image 20260807111240](../assets/Pasted%20image%2020260807111240.png)

![Pasted image 20260807111254](../assets/Pasted%20image%2020260807111254.png)


![Pasted image 20260807111335](../assets/Pasted%20image%2020260807111335.png)

![Pasted image 20260807111540](../assets/Pasted%20image%2020260807111540.png)


![Pasted image 20260807111904](../assets/Pasted%20image%2020260807111904.png)

![Pasted image 20260807112335](../assets/Pasted%20image%2020260807112335.png)


for replace, the syntax is `stringVarName/word_to_replace/word_to_replace_with`

for slice, you provide the length.





![Pasted image 20260807114200](../assets/Pasted%20image%2020260807114200.png)


![Pasted image 20260807114254](../assets/Pasted%20image%2020260807114254.png)


![Pasted image 20260807114433](../assets/Pasted%20image%2020260807114433.png)

![Pasted image 20260807114442](../assets/Pasted%20image%2020260807114442.png)

This does not work!

![Pasted image 20260807114458](../assets/Pasted%20image%2020260807114458.png)

![Pasted image 20260807114523](../assets/Pasted%20image%2020260807114523.png)

![Pasted image 20260807114609](../assets/Pasted%20image%2020260807114609.png)


![Pasted image 20260807114657](../assets/Pasted%20image%2020260807114657.png)

You can also use arithemetic expressions.

![Pasted image 20260807114736](../assets/Pasted%20image%2020260807114736.png)

But it doesn't work lol (double quotes)

![Pasted image 20260807114820](../assets/Pasted%20image%2020260807114820.png)

(Add a `$` sign)

![Pasted image 20260807115313](../assets/Pasted%20image%2020260807115313.png)

![Pasted image 20260807115326](../assets/Pasted%20image%2020260807115326.png)

 ![Pasted image 20260807120111](../assets/Pasted%20image%2020260807120111.png)

Using double brackets is better.

![Pasted image 20260807120219](../assets/Pasted%20image%2020260807120219.png)

![Pasted image 20260807120245](../assets/Pasted%20image%2020260807120245.png)


![Pasted image 20260807123600](../assets/Pasted%20image%2020260807123600.png)

![Pasted image 20260807123906](../assets/Pasted%20image%2020260807123906.png)

![Pasted image 20260807123951](../assets/Pasted%20image%2020260807123951.png)


![Pasted image 20260807124258](../assets/Pasted%20image%2020260807124258.png)

**Always use == for string comparisons**

![Pasted image 20260807124435](../assets/Pasted%20image%2020260807124435.png)

This is ternary!

![Pasted image 20260807124533](../assets/Pasted%20image%2020260807124533.png)


![Pasted image 20260807131025](../assets/Pasted%20image%2020260807131025.png)

![Pasted image 20260807131154](../assets/Pasted%20image%2020260807131154.png)


![Pasted image 20260807134254](../assets/Pasted%20image%2020260807134254.png)

![Pasted image 20260807134724](../assets/Pasted%20image%2020260807134724.png)



![Pasted image 20260807134808](../assets/Pasted%20image%2020260807134808.png)


![Pasted image 20260807134957](../assets/Pasted%20image%2020260807134957.png)


![Pasted image 20260807135026](../assets/Pasted%20image%2020260807135026.png)

![Pasted image 20260807135228](../assets/Pasted%20image%2020260807135228.png)

![Pasted image 20260807135731](../assets/Pasted%20image%2020260807135731.png)

This is also possible


![Pasted image 20260807135748](../assets/Pasted%20image%2020260807135748.png)

![Pasted image 20260807140929](../assets/Pasted%20image%2020260807140929.png)

![Pasted image 20260807141026](../assets/Pasted%20image%2020260807141026.png)

![Pasted image 20260807141125](../assets/Pasted%20image%2020260807141125.png)

![Pasted image 20260807141434](../assets/Pasted%20image%2020260807141434.png)

(alt way)

![Pasted image 20260807142332](../assets/Pasted%20image%2020260807142332.png)

![Pasted image 20260807142743](../assets/Pasted%20image%2020260807142743.png)






