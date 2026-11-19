![](../assets/Pasted%20image%2020261119102734.png)

 Expanding a macro - Replace each macro instruction with the corresponding group of source language statements.

![](../assets/Pasted%20image%2020261119103014.png)

![](../assets/Pasted%20image%2020261119103418.png)

**A macro processor is not directly related to the architecture of the computer on which it is to run**


## Basic Macro Processor Functions

![](../assets/Pasted%20image%2020261119103826.png)

![](../assets/Pasted%20image%2020261119103847.png)


### Macro definition

![](../assets/Pasted%20image%2020261119104007.png)

### Macro Expansion

Each macro invocation statement gets expanded into the statements that form the body of the macro, with the arguments from the macro invocation substituted for the parameters in the prototype.

Copy code:

![](../assets/Pasted%20image%2020261119104613.png)

Parameter substitution:

![](../assets/Pasted%20image%2020261119105253.png)

### Macro Invocation

![](../assets/Pasted%20image%2020261119104714.png)


## Example 1

![](../assets/Pasted%20image%2020261119110125.png)

The expanded code will never contain the macro definition. What happens is though, the macro gets expanded at each line it is invoked.

![](../assets/Pasted%20image%2020261119110501.png)

## Example 2

![](../assets/Pasted%20image%2020261119110740.png)
![](../assets/Pasted%20image%2020261119110750.png)

The main program:

![](../assets/Pasted%20image%2020261119110957.png)

There are 3 macro invocations, so 3 invocations are needed.

Expanded:

![](../assets/Pasted%20image%2020261119111144.png)

Others are expanded similarly.

![](../assets/Pasted%20image%2020261119111354.png)


## Macroprocessor algorithm and data structures

![](../assets/Pasted%20image%2020261119111553.png)

![](../assets/Pasted%20image%2020261119112142.png)


So, we must use a one pass macroprocessor.

### One pass macroprocessor ftw

![](../assets/Pasted%20image%2020261119112339.png)

#### Data structures

3 data structures are used: DEFTAB, NAMTAB, ARGTAB.

![](../assets/Pasted%20image%2020261119112638.png)

DEFTAB stores definitions of macros
NAMTAB stores (name of the macro, beginning of macro definition, end of macro definition)
ARGTAB stores arguments to a macro

![](../assets/Pasted%20image%2020261119113102.png)


#### Algorithm

MAIN program:

![](../assets/Pasted%20image%2020261119113310.png)

Procedure PROCESSLINE:

![](../assets/Pasted%20image%2020261119113352.png)

when opcode is macro, it means add macro to deftab and namtab

Procedure DEFINE:

![](../assets/Pasted%20image%2020261119113413.png)

![](../assets/Pasted%20image%2020261119115305.png)

Procedure EXPAND:

![](../assets/Pasted%20image%2020261119113432.png)

Procedure GETLINE:

if expanding a macro, get the line from the deftab.
else get the line from the asm file.

![](../assets/Pasted%20image%2020261119113459.png)


![](../assets/Pasted%20image%2020261119113535.png)

