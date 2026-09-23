![Pasted image 20260625021231](../assets/Pasted%20image%2020260625021231.png)

the start ( * ) is expanded into the contents of directory

### Pathname expansion

The mechanism by which wildcards work is called pathname expansion.

![Pasted image 20260625021501](../assets/Pasted%20image%2020260625021501.png)
![Pasted image 20260625021510](../assets/Pasted%20image%2020260625021510.png)


### Tilde expansion

![Pasted image 20260625021535](../assets/Pasted%20image%2020260625021535.png)


### Arithmetic expansion

![Pasted image 20260625021608](../assets/Pasted%20image%2020260625021608.png)

![Pasted image 20260625021625](../assets/Pasted%20image%2020260625021625.png)

![Pasted image 20260625021634](../assets/Pasted%20image%2020260625021634.png)


### Brace expansion

![Pasted image 20260625021726](../assets/Pasted%20image%2020260625021726.png)
![Pasted image 20260625021734](../assets/Pasted%20image%2020260625021734.png)
![Pasted image 20260625021744](../assets/Pasted%20image%2020260625021744.png)
![Pasted image 20260625021753](../assets/Pasted%20image%2020260625021753.png)
![Pasted image 20260625021801](../assets/Pasted%20image%2020260625021801.png)


### Parameter expansion

![Pasted image 20260625021906](../assets/Pasted%20image%2020260625021906.png)


### Command substitution

![Pasted image 20260625022010](../assets/Pasted%20image%2020260625022010.png)



## Quoting

### Double quoting

-> If we place text inside double quotes, all the special characters used by the shell lose their special meaning and are treated as ordinary characters.

-> The exceptions are $, \ (backslash), and ` back quote

-> word-splitting, pathname expansion, tilde expansion, and brace expansion are suppressed

-> parameter expansion, arithmetic expansion, and command substitution are still carried out.


### Single quoting

If we need to suppress all expansions, we use single quotes.

![Pasted image 20260625022544](../assets/Pasted%20image%2020260625022544.png)


### Escape characters

For quoting a single char.

![Pasted image 20260625022626](../assets/Pasted%20image%2020260625022626.png)

