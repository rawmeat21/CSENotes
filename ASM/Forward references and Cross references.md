### 1. Forward References

A **forward reference** occurs when an instruction refers to a symbol (such as a branch label or data operand) before that symbol has been defined in the source program. 

**Handling in Two-Pass Assemblers:** Two-pass assemblers resolve forward references in two passes. **Pass 1** scans the source program, tracks addresses via the Location Counter (`LC`), and enters every symbol and its defined address into the Symbol Table (`SYMTAB`). **Pass 2** then generates object code using the completed `SYMTAB`, where all forward-referenced symbols already have known addresses.

**Handling in One-Pass Assemblers:** Look into `One pass assembler.md`


### 2. Cross References

A **cross reference directory** is a report produced by the assembler that lists every reference made to each symbol in a program, sorted in ascending order by statement number. 

It assists in program debugging, maintenance, and reporting errors when a symbol's later definition conflicts with how it was earlier referenced.

**Purpose:**

- **Debugging & Maintenance:** Helps programmers trace every location where a variable is modified or a subroutine/label is jumped to, especially in large multi-routine assembly programs.
    
- **Detecting Unused or Misused Symbols:** Highlights dead variables/labels (defined but never referenced) or unintended references to shared symbols.

**How the Assembler Constructs It:**

- During assembly, whenever a symbol is defined or used as an operand, the assembler records the current source **statement number** alongside that symbol's entry in `SYMTAB` (typically stored as an ordered list or appended sequentially as lines are scanned).
    
- Because the source file is read sequentially from top to bottom, statement numbers are naturally collected in ascending order. At the end of assembly, the assembler sorts `SYMTAB` (usually alphabetically by symbol name) and prints each symbol's definition location followed by its ascending list of reference line numbers.


Example:
```
Line  Loc   Label   Opcode  Operand   Object Code
----  ----  ------  ------  -------   -----------
 05   1000  PROG    START   1000
 10   1000  FIRST   LDA     ALPHA     00100F   (ALPHA: forward ref)
 15   1003  LOOP    ADD     ONE       181012   (ONE:   forward ref)
 20   1006          COMP    ALPHA     28100F   (ALPHA: forward ref)
 25   1009          JEQ     LOOP      301003   (LOOP:  backward ref)
 30   100C          STA     ALPHA     0C100F   (ALPHA: forward ref)
 35   100F  ALPHA   RESW    1         -
 40   1012  ONE     WORD    1         000001
 45   1015          END     FIRST
```


Each symbol records its defined address and links every line number where it is referenced in ascending order:

```
SYMTAB (Symbol | Addr | Ref Ptr)         Reference Nodes (Stmt #)
+--------+------+-----+
| PROG   | 1000 |  /  |
+--------+------+-----+      +----+---+
| FIRST  | 1000 |  *--+----->| 45 | / |
+--------+------+-----+      +----+---+
| LOOP   | 1003 |  *--+----->| 25 | / |
+--------+------+-----+      +----+---+      +----+---+      +----+---+
| ALPHA  | 100F |  *--+----->| 10 | *-+----->| 20 | *-+----->| 30 | / |
+--------+------+-----+      +----+---+      +----+---+      +----+---+
| ONE    | 1012 |  *--+----->| 15 | / |
+--------+------+-----+      +----+---+
```

The final report looks like this:

```
Symbol    Address   Defined (Line)   Referenced (Lines)
------    -------   --------------   ------------------
ALPHA     100F      35               10, 20, 30
FIRST     1000      10               45
LOOP      1003      15               25
ONE       1012      40               15
PROG      1000      05               -
```