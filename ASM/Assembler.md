![](../assets/Pasted%20image%2020261019152808.png)

Assembler is the tool to convert an assembly language program into machine language one, understandable by the processor that executes it.

## Functions

 -> Translating mnemonic operation codes to their machine language equivalents
 -> Convert symbolic (e.g., jump labels, variable names) operands to their machine addresses.
 -> Use proper addressing modes and formats to build efficient machine instructions
 -> Translate data constants into internal machine representations
 -> Output the object program and provide other information (e.g., for linker and loader)

![](../assets/Pasted%20image%2020261019153953.png)


The assembler keeps track of two kinds of symbols:

- **User-defined symbols** are names you define in your own program, such as labels and variables. The assembler can resolve these itself, because it sees where they are defined.
- **External symbols** are names used in your program but defined elsewhere, such as a subroutine in another source file or a library. The assembler cannot know their addresses, because it only sees one program at a time.

The assembler passes these external symbols to the loader and linker. **Linker/loader**, combines the separately assembled modules. It finds the program that defines the external symbol and fills in the real address afterward.

![](../assets/Pasted%20image%2020261020110655.png)

- **Input:** the user’s assembly language program.
- **Process:** the assembler, which has two parts:
    - **Analysis** reads the source and understands each statement: which mnemonic it is, which operands it has, and which symbols are defined.
    - **Synthesis** builds the machine-language output from what analysis found.
- **Databases** that both parts consult:
    - **Machine Operation Table (MOT):** mnemonic to machine opcode, with instruction length and format.
    - **Pseudo Operation Table (POT):** assembler directives such as `START`, `END` and `DS`. These give instructions to the assembler and do not become machine instructions.
    - **Symbol Table:** each symbol and its address.
    - **Literal Table:** constants written directly in the code, such as `=5`, and where they are stored.
- **Output:**
    - **Object code**, the translated machine code.
    - **Linker information**, the external-symbol and fix-up data from the bullets above.

## Limitations of assembler

![](../assets/Pasted%20image%2020261020110953.png)


## Assembler directives

Assembler directives are pseudo instructions. They are not translated into machine instructions. They only provide instruction/direction/information to the assembler.

Some assembler directives:

- START : Specify name and starting address for the program. `COPY   START  1000` means program is named `COPY` and the first byte is at address `1000`.
- END : Indicate the end of the source program, and (optionally) the first executable instruction in the program.
- BYTE :  Generate character or hexadecimal constant, occupying as many bytes as needed to represent the constant.
- WORD : Generate one-word integer constant
- RESB : Reserve the indicated number of bytes for a data area
- RESW : Reserve the indicated number of words for a data area

