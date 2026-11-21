**Overlaying** is a memory-management and program-structuring technique used when the total size of an executable program exceeds the available main memory (for example, on a standard SIC machine with only 32 KB32 KB, or 215215 bytes, of memory). 

Instead of loading the entire program at once, the programmer or system divides the program into a permanently resident **root segment** and multiple **mutually exclusive overlay segments** that share the same physical memory region at different points during execution.


## Normal Loading and Overlay Loading

- In **normal loading**, all control sections and subroutines required by the program are linked and loaded into contiguous or separate non-overlapping memory locations simultaneously before execution begins.
    
- In **overlay loading**, only the **root segment** (containing the main driver routine, common global data, and the overlay supervisor/manager) remains permanently resident in memory. Mutually exclusive routines are assigned to the same physical memory area and overwrite one another as execution progresses.


## Overlay Tree Structure

An **overlay tree** expresses the logical mutual exclusion among program modules.

- **Root Node:** Placed at the top of the tree, remains resident throughout program execution.

- **Sibling Nodes:** Child nodes that are never resident simultaneously and can therefore reuse the same physical memory region.
    
- **Nested Overlays:** Deeper levels of the tree define nested reusable regions allocated immediately after their parent overlay's end address.

The loader/linker must know where each overlay belongs.

![](../assets/Pasted%20image%2020261120233541.png)

### Memory calculation

![](../assets/Pasted%20image%2020261120233612.png)

The minimum main memory required to execute an overlay-structured program is determined by the **longest root-to-leaf path** in the overlay tree.


## Role of assembler

-> The assembler translates each module as a separate **control section** (`CSECT`), resetting the Location Counter (`LC`) to `0000` for each section and building an independent symbol table (`SYMTAB`) for local labels. (In SIC/XE, the `USE` directive can also partition code/data into program blocks prior to linking.)

-> **External Interface Records (`D` and `R`):**

- `EXTDEF` and Define records are used to define symbols within the overlay that other modules may call.
- `EXTREF` and Refer records are used to refer symbols defined in some other control section.

-> The assembler emits **Modification (`M`) records** identifying every address field that must be adjusted upon loading.

Roles in short:

1. Translate each module into object code.
2. Build symbol tables and resolve local labels.
3. Track control sections and external definitions/references.
4. Generate relocation/modification information for later placement.
5. Overlay placement is coordinated with linking/loading rather than solved by mnemonic translation alone.


## Role of loader

- Combines separately assembled control sections, enters all `EXTDEF` symbols into the External Symbol Table (`ESTAB`), and resolves `EXTREF` dependencies.
    
- Using the overlay tree specification, the linker assigns the root segment at address `PROGADDR` and assigns each group of mutually exclusive sibling overlays the **same starting control-section address (`CSADDR`)**, sizing each region to max⁡(sibling sizes).
    
Roles in short:

1. Combine separately assembled modules.
2. Resolve EXTDEF/EXTREF symbols.
3. Assign addresses to control sections and overlay regions.
4. Apply relocation using modification information.
5. Prepare the runtime layout expected by the overlay manager.


## Role of linker

- Loads the root segment and the initial active overlay path into memory while protecting the root segment and persistent global data from being overwritten.
    
- When a routine in the root (or an active parent overlay) invokes a routine residing in an overlay segment, control passes through the **Overlay Manager**:
    
    1. **REQUEST:** The caller requests a target routine/overlay.
        
    2. **CHECK RESIDENT:** The overlay manager inspects the Overlay Table to see if the target overlay is already `RESIDENT` in its designated region.
    
    3. **LOAD:** If absent, the manager reads the overlay's object module from secondary storage into the reusable memory region, replacing any conflicting sibling overlay.
        
    4. **RELOCATE:** If the region supports dynamic placement, any remaining `M` records are applied relative to the region's `CSADDR`.
        
    5. **TRANSFER CONTROL:** Residency status is updated to `ACTIVE`, and execution branches to the target routine's entry point.

Roles in short:

1. Load the root and initial overlay.
2. When another overlay is needed, place it in the reusable region.
3. Perform required relocation.
4. Update residency information and transfer control only after loading completes.
5. Protect root code and persistent data from overlay replacement.

![](../assets/Pasted%20image%2020261120235017.png)



## Control sections in Overlays

1. Control sections support separately assembled logical modules.
2. EXTDEF exports symbols defined in a section.
3. EXTREF identifies symbols defined elsewhere.
4. Modification records identify address fields needing adjustment.
5. These mechanisms form the object-program foundation for modular overlay loading.


## Relocation in Overlays

1. An overlay may load at an address different from its assembly-time address.
2. Absolute address fields may need modification.
3. PC-relative/base-relative addressing can reduce relocation when the target fits the number of bits used for addressing. For such addressing, no modification records are required as these are relative addresses.
4. Format-4 style extended addresses may require relocation metadata. Calls between the root and an overlay, cross-overlay references, or address constants (`WORD`) contain full 20-bit addresses. These require **Modification (`M`) records**.
5. Correct relocation is essential when a shared region is reused.


## Modification records in Overlays

1. A modification record identifies the object-code field to change.
2. It carries the starting location and field length, commonly measured in half-bytes.
3. External-symbol information can specify what value is added/subtracted.
4. Overlay modules must retain valid relocation metadata for their eventual load address.


## Two pass assembly

1. Pass 1: assign locations, collect symbols, literals and control-section information.
2. Pass 2: encode instructions and emit object, external-reference and relocation records.
3. Link/load stage maps modules into the overlay structure.
4. This separation makes address resolution and final placement easier to manage.


## One pass assembly

1. Forward references require backpatching or forward-reference chains.
2. External symbols cannot always be finalized during the first scan.
3. Overlay linking increases the number of dependencies between modules.
4. Literal and relocation handling must remain correct when modules move.


## Overlay Memory optimization

1. Group mutually exclusive routines into the same reusable region.
2. Size each region for its largest member.
3. Keep frequently used/shared routines resident when beneficial.
4. Avoid sharing storage between data objects whose lifetimes overlap.
5. Minimize overlay transitions on hot execution paths.


## Performance and Tradeoff

1. Benefit: lower peak memory footprint.
2. Cost: loading I/O and relocation overhead at overlay boundaries.
3. Too many small overlays can cause excessive loading.
4. Too few large overlays increase memory requirements.
5. Good overlay design balances resident size against transition frequency.


## Comparison with paging

1. Overlaying is usually explicit and program-structured. Paging is generally managed by the operating system/hardware.
2. Overlays exploit known mutual exclusion; paging exploits runtime locality.
3. Overlaying is especially useful where virtual memory is absent or constrained.
4. Both retain only a working subset in memory.


## Comparison with Dynamic linking

1. Overlaying focuses on memory-region reuse and residency. Dynamic linking focuses on resolving/loading reusable modules or libraries. These techniques can coexist.
2. An overlay region can contain code supplied by separately linked modules.



![](../assets/Pasted%20image%2020261121000308.png)


![](../assets/Pasted%20image%2020261121000320.png)








