A **hex dump** is a textual display of raw binary data stored in memory or a file.

Because standard text editors treat arbitrary bytes as ASCII/UTF-8 text, opening a raw binary file directly renders garbage characters, executes control codes, or breaks the terminal. A hex dump translates every single byte into its 2-digit hexadecimal value (`00` through `FF`) and displays it in a structured grid.


```
Offset Address    Hexadecimal Byte Values (16 bytes per line)      ASCII Representation
|-------| |-----------------------------------------------|        |---------|
00000000       01 c8 d6 03 e8 fc 00 1a  2b 3c 4d c1 48 00 00 de    |........+<M.H...|
00000010       ad be ef 43 4f 44 45                               |...CODE|
```

