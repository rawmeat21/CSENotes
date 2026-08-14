When not to use shell scripts

- Resource-intensive tasks, especially where speed is a factor (sorting, hashing, recursion [[2]](https://tldp.org/LDP/abs/html/why-shell.html#FTN.AEN87) ...)
    
- Procedures involving heavy-duty math operations, especially floating point arithmetic, arbitrary precision calculations, or complex numbers (use _C++_ or _FORTRAN_ instead)
    
- Cross-platform portability required (use _C_ or _Java_ instead)
    
- Complex applications, where structured programming is a necessity (type-checking of variables, function prototypes, etc.)
    
- Mission-critical applications upon which you are betting the future of the company
    
- Situations where _security_ is important, where you need to guarantee the integrity of your system and protect against intrusion, cracking, and vandalism
    
- Project consists of subcomponents with interlocking dependencies
    
- Extensive file operations required (_Bash_ is limited to serial file access, and that only in a particularly clumsy and inefficient line-by-line fashion.)
    
- Need native support for multi-dimensional arrays
    
- Need data structures, such as linked lists or trees
    
- Need to generate / manipulate graphics or GUIs
    
- Need direct access to system hardware or external peripherals
    
- Need port or [socket](https://tldp.org/LDP/abs/html/devref1.html#SOCKETREF) I/O
    
- Need to use libraries or interface with legacy code
    
- Proprietary, closed-source applications (Shell scripts put the source code right out in the open for all the world to see.)