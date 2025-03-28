Status: #inprogress
Jochen Liedtke, SIGOPS 1995
[Paper](https://dl.acm.org/doi/pdf/10.1145/224056.224075)
## Problem Statement
*What issue does the paper address, and is it significant and worth solving? Are the authors solving a real problem or creating one?*

Key Problem Statement: most existing microkernels do not perform sufficiently well, despite technological advantages of the approach. The author challenges readers to consider the *reasons* for poor performance and analyse whether it is due to either the Approach to microkernels, the Concepts implemented by microkernels or the Implementation of microkernels.

The cited technological advantages of a microkernel approach include modular system design, isolating malicious or malfunctioning server components, and flexibility of system design.

It is therefore in the interests of the community to build microkernels that are performant, so it is a worthwhile endeavour to explore the underperformance of existing systems a bit more deeply.
## Related Work
*Are there uncited related publications? Is proper credit given to previous work, and is it accurately represented? Are past lessons correctly presented?*


## Proposed Solution
*Summarise the approach, including system design and algorithms. Does the solution address the problem effectively? Is it novel or previously proposed? Does it improve on previous work? Is it convincing and feasible, with no overlooked challenges? Are alternative approaches discussed?*

The author defines three broad categories of reasons for underperformance of microkernels:
- Approach
- Concepts
- Implementation

### Concepts of a minimal microkernel
In defining minimal concepts, the author requires that *only* that functionality which is *necessarily* implemented in the kernel is done so.

**Address Spaces**
Microkernel abstracts the hardware notion of address spaces (TLB, page tables).

Address space construction should be achievable outside of the kernel. Initial subsystem is given an address space covering the entire physical memory, and with kernel-provided operations can construct other address spaces *from* this initial one (imagine a tree of address spaces, with child nodes being a subset of parent nodes). Operations are on the page level:
- Grant (move from one address space to another).
	- used only in specialised circumstances - the author gives an example of a pager which combines multiple underlying file systems.
	- Does this contradict the principle of minimality? As an additional consideration, it does appear that Map is *sufficient* in any situation Grant would be used, just with additional work e.g. bookkeeping. This doesn't mean it isn't a good idea.
- Map (copy from one address space to another).
- Flush (remove from all other *descendant* address spaces which received page from Grant or Map operations). 

Memory managers require map and flush operations. 

**Threads**
- execution *within* an address space + associated state.
- kernel controls all address space changes.
- communication between threads requires kernel involvement - IPC.

**IPC**
- message-passing between threads, supported by the kernel.
- agreement between sender (what to send) and receiver (willing to receive).
- address space operations Grant/Map provided by IPC (to enforce such agreement).
- Interrupts abstracted as IPC - such transformation is done by the kernel but interrupts are handled in user space.
- No mention of shared memory as an alternative to large IPC messages - another potential violation of minimality principle.

**Unique Identifiers**
- kernel is trusted - so subsystems can trust that the sender/receiver of a message is known and enforced to the extent the kernel keeps track of such IDs.

## Assumptions and Limitations
*What assumptions and limitations are identified, and are they reasonable? Is the solution still useful despite them? Are there any unstated assumptions or hidden limitations? Is the solution credible despite its limitations?*

- Target system supports interactive and/or not completely trustworthy applications.
- Address space concept is illustrated without detailed assumptions like specific access rights.
	- I think this is okay for the purpose of the paper, the author does concede this point and indicates the extension should be reasonably straight forward. For example, one can grant/map specific access rights (but only restrict, not widen, access), and/or provide special flushing operations for specific access rights.
- 
## Evaluation
*How is the solution evaluated, and is the evaluation convincing? Are appropriate metrics used, and does it align with original goals? Is the solution's cost justified? Is the evaluation comprehensive, covering all aspects of the solution? Are any measurements missing? Are weak points, assumptions, and limitations explored? Are complete results presented, avoiding biases and benchmarking crimes? Is the comparison against competing approaches fair? Is sufficient information provided to assess results? Is an appropriate baseline shown, and is important information not hidden by relative numbers?*

- using best-case related work performance metrics for comparison.
## Presentation
*Is the paper well-structured and clear? Are the problem and solution clearly presented with enough detail? Are high-level design choices explained first? Are concepts introduced systematically? Is the reader not overwhelmed with details? Is the presentation of evaluation results sensible? Are there no issues with grammar, spelling, or sloppiness?*