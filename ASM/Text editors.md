#### The Six Core Edit-Control Features

The edit control features are controls available to the user of the editor. Thus, before concentrating on the design of the editor, these features are to be fixed. The features present in most of the modern text editors are:

1. **Unlimited File Size and Line Length Editing:** The edit control should allow editing files of arbitrary size and lines of arbitrary length with little or no degradation in performance as file size grows. (This relies on `TextDocument` managing data in chunks/buffers rather than requiring fixed-width line arrays.)
    
2. **Fast, Smooth Graphic Display:** The display must render text cleanly and support responsive text selection as well as smooth vertical and horizontal scrolling without visual artifacts or glitches.
    
3. **Syntax Colouring:** For program source files, the edit control highlights distinct lexical components of a programming language such as keywords, variables, and constants in different colours at runtime so programmers can quickly identify code structures.
    
4. **Full Clipboard and Drag-and-Drop Support:** Users can select blocks of text and perform standard clipboard operations (cut, copy, and paste) as well as mouse-driven drag-and-drop text movement within a document or across applications.
    
5. **Single- or Multi-Font Display:** The edit control supports displaying text in a single uniform font or mixing multiple fonts of varying widths and heights (aligning characters along a common baseline and adjusting line height to fit the tallest font on that line).
    
6. **Complete Undo and Redo Support:** The editor tracks editing history so users can revert previous insertions, deletions, or modifications (`Undo`) and reapply reverted changes (`Redo`) (typically backed by sequence structures like a piece table).

7.  **Cursor Navigation & Scrolling:** Moving via arrow keys, Page Up/Down, mouse clicks, or go-to-line commands, updating visible line ranges via vertical and horizontal scrollbars.

8. **Core Text Manipulation & Search/Replace:** Inserting, deleting, and replacing characters or marked blocks, alongside pattern searching and single/global substitution.


## Line editors and screen editors

_Editors_ are utilities to write programs in source languages. There are mainly two categories of such editors, namely,

1. line oriented editors
    
2. screen oriented editors.
    

The distinction between the two categories is made based upon the smallest unit that can be edited:

-> In a _line editor_, the unit of editing consists of a full line. That is, to modify even a single character, the user has to modify/re-enter the complete line. These are very primitive type of editors. For example, the editor _edlin_ of _DOS_, _ed_ and _ex_ of _Unix_, etc. 

-> _screen-oriented editors_ allow using the cursor keys and/or mouse to point to a position at which the next editing is to be carried out. All modern systems come up with one/more such screen oriented text editors. In the following, we briefly review some of these:

_vi_: The editor _vi_ was created by _Bill Joy_, cofounder of _Sun_ in 1976. It is a considerably powerful utility. The _vi_ editor works in two modes: the _insert mode_ and the _command mode_. For entering data, the _insert mode_ is to be used, whereas to copy, paste, or generally adjust the file, one should switch over to the _command mode_.

_Emacs_: This is an especially powerful editor because while its core is a plain text editor, it is a whole lot more too. One can add tools to Emacs to help in writing HTML or SGML, and use it to code a variety of languages. Like a lot of open source software, it has plenty of detailed online documentation and modules. One can add other features such as spell check or data encryption, or write own modules. Emacs also has support for different programming languages.

_Notepad_: _Windows_ Notepad offers features such as: the ability to cut, copy, paste, and find (but not replace). One can set fonts and styles within the document, but these are for display only, as the output is plain text, which means no document formatting code is created from within the program.

![](../assets/Pasted%20image%2020261121091653.png)

![](../assets/Pasted%20image%2020261121091842.png)



## Design of a text editor

![](../assets/Pasted%20image%2020261121092803.png)

A text editor consists of a main window along with some edit controls. However, the other part of it is the text file being edited. The actual file resides in the disk in some format that may be totally different from the way it is shown in the edit window. The key components of a text editor are the following:

1. _Main Window_: The top-level main window generally contains the title of the file being edited, menu, and status bars. It does not know how to edit text files, its only purpose is to provide an interface to the user.
    
2. _TextView_: TextView is the most important component of a text editor. It is generally designed as a separate window that is a child of the _Main Window_. The _TextView_ is the visual component of a text editor. Its primary purpose is to display and edit text. However, it needs to do a number of other things, such as displaying scrollbars, process mouse and keyboard inputs, and support drag and drop.
    
3. _TextDocument_: It is not a visible component, but is used to store and manipulate a text file, once it has been loaded into the memory. It does not have any concept of windows, mice, drawing or painting. All that it knows is how to manipulate text, and supply text to the edit control when it wants to draw something.
    
4. _Text File_: Though it is not really a part of the text editor, it plays an important role nonetheless. It is the disk file that stores the text we want to edit. The TextDocument will interface directly with this text file, reading from and writing to it when it is time to load or save a new document.
    

It is important to note that separation of _TextView_ from the data it stores (that is, _TextDocument_) provides the flexibility to modify the way we store and represent the actual data in the text file without impacting upon the design of visual component. 

Similarly, the visual component can be modified without any necessity to modify the storage organization of the text file. The interface between the components may also vary. 

For example, the _TextView_ is a line-oriented, graphical entity. Most of the work performed by the edit-window will be updating the display. This will entail retrieving data from the _TextDocument_ on a line-by-line basis. On the other hand, the _TextDocument_ will either load the entire file into memory in one go, or access the file in manageable chunks.


## Cross assembler

A **cross assembler** is an assembler that executes on one processor architecture (the **host** machine) while translating assembly source code into machine code for a different processor architecture (the **target** machine). It becomes indispensable for system and embedded development for the following reasons:

1. **Resource-Constrained Target Systems Cannot Host Development Tools:** Single-chip microcontrollers ("computers on a chip"), embedded controllers, and specialized target boards have severely limited RAM, storage, and I/O interfaces, making them too small to run text editors, assemblers, linkers, or store large source files locally. Offloading translation to a capable host machine makes software development possible for bare-metal targets.
    
2. **Parallel Software and Hardware Development (Before Hardware Exists):** During new system development, target hardware is often still being designed or fabricated. A cross assembler allows engineers to write, assemble, and verify target code early using an instruction-set simulator on the host PC before the physical target board is ready.
    
3. **Faster Development and Richer Host Environment:** Editing and assembling programs on a fast host workstation is significantly quicker than running tools on a low-powered target processor. Developers gain access to full-featured host editors, version/file management, macro expansion, conditional assembly, and detailed symbol/cross-reference listings.
    
4. **Precise Target Memory Mapping and ROM/Firmware Deployment:** Embedded system software must often be placed at exact ROM and RAM addresses. Cross assemblers support target memory-map directives (such as `ORG`) and emit standardized transfer formats such as **Intel HEX** or **Motorola S-records** that can be downloaded over a serial link into target RAM via a ROM monitor or burned directly into EPROM/ROM.
    
5. **Multi-Variant Firmware Maintenance:** Through conditional assembly directives (`$SET`, `$IF`, `$ENDIF`) and parameterized macros, a cross assembler allows developers to maintain a single assembly codebase on the host and generate tailored machine-code images for multiple target hardware revisions.


### Features of a cross assembler

1. **Pseudo-ops (Assembler Directives):** It provides directives to control target memory layout and data allocation, including `ORG` (set location counter), `EQU` (assign a symbolic constant), `DS` (reserve storage), and `DC.B`/`DC.W` (define byte or word constants).

2. **Conditional Assembly:** Using directives such as `$SET`, `$IF`, `$ELSEIF`, and `$ENDIF`, it allows selective assembly of code blocks so a single source file can produce firmware for multiple target hardware configurations.
    
3. **Macro Processing:** It supports macro definitions (`$MACRO ... $MACROEND`) with positional argument substitution (`%n` for the n-th parameter). It also generates unique labels automatically to prevent duplicate-symbol conflicts.
    
4. **Two-Pass Translation Structure:** Like native assemblers, it typically uses a two-pass design where Pass 1 builds the symbol table and Pass 2 generates the object code and listing.
    
5. **Target-Specific Output Formats:** Along with an assembler listing (location counter, object code, and source statements), symbol table, and cross-reference table, it emits downloadable target formats such as **Motorola S-records** or **Intel HEX**.
    
6. **Absolute and Relocatable Assembly Modes:** It supports both absolute mode (where `ORG` fixes physical addresses in a single source file) and relocatable mode (where separate modules are linked and mapped to target ROM and RAM by a linker/locator).


