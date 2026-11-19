## Machine independent macro processor features

![](../assets/Pasted%20image%2020261119115547.png)


### Concatenation of macro parameters

![](../assets/Pasted%20image%2020261119133354.png)

![](../assets/Pasted%20image%2020261119133536.png)

![](../assets/Pasted%20image%2020261119133738.png)

![](../assets/Pasted%20image%2020261119133920.png)


### Generation of unique labels

![](../assets/Pasted%20image%2020261119134123.png)

When the above macro is expanded 2 times, the label LOOP gets duplicated and points to 2 locations. This causes the assembler to give an error when trying to convert the program file to object code.

**Solution 1: use relative addressing**

![](../assets/Pasted%20image%2020261119134625.png)

In the expanded macro, we don't use any labels. Instead, we use relative addresses.

`JLT *-11` jumps `-11` bytes from where the LOCCTR is. Why 11? Basically, `JLT *-11` is at  0015. LC=0018 after this. LC was 000D at `LDCH STR1,X`. We should jump to 000D, so subtract 000D from 00018: `0018-000D=24-13=11`.

![](../assets/Pasted%20image%2020261119135434.png)


**Solution 2: unique labels**

![](../assets/Pasted%20image%2020261119135526.png)

![](../assets/Pasted%20image%2020261119135647.png)
![](../assets/Pasted%20image%2020261119135734.png)


![](../assets/Pasted%20image%2020261119135758.png)


### Conditional macro expansion

Most macro processors can modify the sequence of statements generated for a
macro expansion, depending on the arguments supplied in the macro invocation.

#### Macro processor directives

IF, ELSE, ENDIF, SET.

![](../assets/Pasted%20image%2020261119140611.png)

At the very top, `&EORCK` is set to 1 if `&EOR` is not empty string.

After that if `&EORCK` is equal to 1, then the block after `IF` gets exectued upto `ENDIF`.

Below, you can also see a `IF-ELSE` construct.

Here is an example expansion:

![](../assets/Pasted%20image%2020261119141232.png)


#### Macro time variables

Using SET symbol.


Also, we have WHILE, ENDW for looping.


### Keyword macro parameters

#### Positional parameters

![](../assets/Pasted%20image%2020261119142251.png)

#### Keyword parameters

![](../assets/Pasted%20image%2020261119142521.png)

example:

![](../assets/Pasted%20image%2020261119142729.png)

In the above macro, default paramters are specified using concept of keyword parameters. We don't set default values for BUFADR and RECLTH. 

While calling, we are only required to specify the values for BUFADR and RECLTH:

![](../assets/Pasted%20image%2020261119142845.png)

However, we can provide more:

![](../assets/Pasted%20image%2020261119143125.png)
