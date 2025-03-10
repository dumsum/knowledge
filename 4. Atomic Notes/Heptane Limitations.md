- Loop bound annotations are completely manual.
- minor: needed some tweaks to correctly detect function vs instruction in the `objdump` output
- minor: other binary sections will be parsed but only the `.text` section is analysed for CFG extraction - so functions from other sections (e.g. `.boot.text`) pollute the output
- instructions are parsed in code - not all are present and it does not gracefully handle unknowns
	- e.g. needed to tell it about `sret`, `snez`, various `csr`, `sc.w`, `ecall`, `ebreak`, `rdtime` 
	- CFG extractor seems to need information about instructions to determine how to build Basic Blocks (e.g. needs to differentiate conditional/unconditional jumps, calls, returns etc.)
- instruction *format* missing: `sc.w zero,zero,(t0)`
- the loop detection/analysis component doesn't play well with jump tables and will incorrectly detect loops where there are none - pertinent example is `sameRegionAs` 
	- I guess they sort of behave like indirect branches so can understand the difficulty but all information should be known at compile time 
	- Building seL4 with `-fno-jump-tables` helps - I have fairly reasonable looking CFGs from that. 
- Tail Calls aren't handled well - it doesn't draw edges from one CFG to another (each function has a CFG).
	- example is `trap_entry` etc. which just jump to other parts of code before execution eventually hits `sret` for example. 
	- could potentially be addressed by combining such CFGs - not sure if the provided library can handle this easily.
	- WCET analysis seems to skip over CFGs that don't have an "end node" (BB with return) so this does become an issue.
- Assumes 32-bit instruction length, causing issues with BB identification and control flow analysis because seL4 builds with the C-extension by default.
    - Mitigated by compiling without the extension, resulting in a larger binary but solving spurious loop problem.
- Assumes functions don't modify the stack pointer (except initial stack frame allocation), which is problematic from a kernel perspective.
    - May try starting only from C functions and manually analyse ASM setup code.
- Doesn't consider CSR instructions and registers. Manually added but needs further investigation.
- Data Cache Analysis: Requires simulation step associating memory addresses to load/store instructions. Heptane currently only caters for stack-relative addresses and loads from global pointer relative addresses. Many other register operations not implemented (with a smattering of "NYI" and "TODO" comments through the codebase).
    - Overall, currently incomplete, effectively unusable for data cache analysis.
- Data cache analysis is ignored in Pipeline analysis, confirmed by a log message present in the codebase.
- Pipeline analysis is hardcoded at 4 stages (Fetch, Decode, Execute, Write Back). Flexibility for other designs would be useful.
    - "Memory access" stage of the "Classic RISC pipeline" is not implemented, flagged with `TODO`. This might be where a DCache miss would be accounted for with a pipeline stall.
- DCache analysis can be considered without Pipeline analysis, but I assume both required for our purposes.
- No support for superscalar or out-of-order execution.
- No SMP support.