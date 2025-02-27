[Paper](https://trustworthy.systems/publications/nicta_full_text/4863.pdf)
_IEEE Real-Time Systems Symposium_, pp. 339–348, Vienna, Austria, November, 2011

Status: #inprogress
## Problem Statement
*What issue does the paper address, and is it significant and worth solving? Are the authors solving a real problem or creating one?*

The paper is attempting to establish Worst Case Execution Time (WCET) of an OS kernel and claims to be the first such analysis of "an OS kernel providing full virtual memory and memory protection".

It introduces the notion of a "hard real-time system": a system whose interrupt response time requires a well-established upper bound.

Traditional approaches to designing such a system include separating out the timing critical components onto designated processors, but cost can add up quickly. By mixing critical and non-critical components onto the one processor (in particular, commodity hardware), cost is kept down but timing guarantees may be weakened.

WCET analysis of the OS can help establish these guarantees regardless of system state at the time of an interrupt. This has typically not been possible due to code complexity and size, but the design of seL4 facilitates such an analysis. 

The contribution, therefore, is to demonstrate seL4's suitability as a "platform for safety- and timing-critical systems", at a significantly lower cost than traditional approaches. Novelty is exhibited through the claim of being the first complete WCET analysis of an OS providing full memory protection. There is clear utility for embedded system designers who could use seL4 as a basis for incorporating enhanced functionality (such as user convenience) alongside the hard real-time requirements of systems without excessive cost.
## Related Work
*Are there uncited related publications? Is proper credit given to previous work, and is it accurately represented? Are past lessons correctly presented?*

Some related work is credited but not analysed in detail. This is perhaps due to the difficulty of the problem in analysing WCET for an OS kernel, particularly one providing full memory protection.
- all systems previously analysed are for "unprotected, single-mode kernels"
- a failed attempt for L4 Pistachio was previously conducted

This appears to be a niche space and so the absence of detailed review of prior work is acceptable.
## Proposed Solution
*Summarise the approach, including system design and algorithms. Does the solution address the problem effectively? Is it novel or previously proposed? Does it improve on previous work? Is it convincing and feasible, with no overlooked challenges? Are alternative approaches discussed?*
### Pre-emption in the kernel
The paper makes the observation that interrupt latency can be greatly reduced by making a fully pre-emptible kernel. This gives rise to concurrency issues and makes verification of functional correctness difficult. 

Conversely, seL4 executes with interrupts disabled and has checks for pending interrupts at specified points. This simplifies behavioural analysis at the expense of a greater interrupt latency, however it is suggested that this is less of a concern with fast, modern processors.

Thus seL4 admits a tuneable trade-off between number of "pre-emption points" and latency.
### Details
Calculating the WCET is done with an seL4 binary compiled with GCC, `-O2` optimisation and `-fwhole-program` flag.
- aggressive inlining means the compiled code doesn't correlate well with the C source
- Chronos apparently has a feature to assist with this correlation

Absence of function pointers means a CFG of seL4 can be extracted without user guidance.

The paper uses symbolic execution which is required because of evaluation of pointer arithmetic on R-W memory such as return addresses on the stack. 
## Assumptions and Limitations
*What assumptions and limitations are identified, and are they reasonable? Is the solution still useful despite them? Are there any unstated assumptions or hidden limitations? Is the solution credible despite its limitations?*
## Evaluation
*How is the solution evaluated, and is the evaluation convincing? Are appropriate metrics used, and does it align with original goals? Is the solution's cost justified? Is the evaluation comprehensive, covering all aspects of the solution? Are any measurements missing? Are weak points, assumptions, and limitations explored? Are complete results presented, avoiding biases and benchmarking crimes? Is the comparison against competing approaches fair? Is sufficient information provided to assess results? Is an appropriate baseline shown, and is important information not hidden by relative numbers?*
## Presentation
*Is the paper well-structured and clear? Are the problem and solution clearly presented with enough detail? Are high-level design choices explained first? Are concepts introduced systematically? Is the reader not overwhelmed with details? Is the presentation of evaluation results sensible? Are there no issues with grammar, spelling, or sloppiness?*