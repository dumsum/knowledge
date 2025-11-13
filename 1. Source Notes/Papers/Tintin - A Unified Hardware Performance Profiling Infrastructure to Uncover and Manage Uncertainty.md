[Paper](https://www.usenix.org/system/files/osdi25-li.pdf)
Ao Li, Marion Sudvarg, Zihan Li, Sanjoy Baruah, Chris Gill, Ning Zhang
Washington University in St. Louis 
OSDI 2025
## Problem Statement
*What issue does the paper address, and is it significant and worth solving? Are the authors solving a real problem or creating one?*

Hardware Performance Counters capture low‑level microarchitectural events critical for understanding program behaviour. Two structural problems persist:

1. **Measurement** – There are far more events of interest than hardware counters available, so events must be multiplexed. This creates sampling gaps and statistical error.
2. **Attribution** – Profiling needs differ (per‑thread, per‑core, per‑region, cross‑cutting), but current OS APIs force one‑size‑fits‑all scope control. Statistical sampling approach: can record program counter and map back but incurs sampling bias in favour of frequently executed code.

**Pro** – Highlights genuinely distinct bottlenecks: one in raw measurement fidelity, the other in the semantics of _where_ and _how_ profiling is applied. These resonate with both kernel/tooling developers and performance engineers. 

**Skeptical** – The severity of multiplexing error may be overstated for broader use‑cases (e.g. coarse regressions, high‑level trends). Scope‑flexibility problems may primarily impact a subset of advanced users, making this a specialised rather than universal pain point.
## Related Work
*Are there uncited related publications? Is proper credit given to previous work, and is it accurately represented? Are past lessons correctly presented?*

Mainstream tools (`perf_event` et al.) avoid counter starvation via multiplexing but do not quantify or actively minimise the induced error; scope control is largely tied to fixed abstractions. Earlier research treats one side of the problem or uses bespoke, non‑general frameworks.

**Pro** – Clear gap analysis — surveying both production and academic tooling — helps justify why a unified solution is non‑trivial.

**Skeptical** – By compressing the prior‑art landscape, the paper risks glossing over partial solutions (e.g. domain‑specific profilers, vendor‑specific enhancements) that have elements of uncertainty modelling or flexible scoping.
## Proposed Solution
*Summarise the approach, including system design and algorithms. Does the solution address the problem effectively? Is it novel or previously proposed? Does it improve on previous work? Is it convincing and feasible, with no overlooked challenges? Are alternative approaches discussed?*

Tintin combines:

1. **Runtime uncertainty characterisation** – quantifies multiplexing‑induced error on the fly. Variance in event counts is used as a proxy for this uncertainty. 
2. **Uncertainty‑aware scheduling** – prioritises events to minimise expected error given counter limits.
3. **Reporting** – makes error estimates visible to applications. Adds an **Event Profiling Context (ePX)** primitive for composable, cross‑cutting scope definitions beyond process/thread/core. Compatible with existing `perf_event` APIs for gradual adoption.

**Pro** – Bridges low‑level kernel mechanisms and high‑level API usability in a coherent architecture; offers a potentially reusable kernel primitive (ePX) beyond this specific project.

**Skeptical** – Runtime modelling overhead may not be negligible at scale; scheduler and API integration require non‑trivial kernel changes, which can stall real‑world uptake. “Drop‑in” compatibility may hide migration and tooling costs.
## Assumptions and Limitations
*What assumptions and limitations are identified, and are they reasonable? Is the solution still useful despite them? Are there any unstated assumptions or hidden limitations? Is the solution credible despite its limitations?*

Assumes:

- Multiplexing error has predictable structure that can be modelled online.
- Applications benefit meaningfully from uncertainty data.
- Kernel maintainers are willing to integrate ePX and related scheduler changes. Evaluation is bound to a set of machines and workloads.

Limitations:

- Scalability limit of 512 concurrent events (minor)

**Pro** – Focused assumptions simplify experimentation and allow strong quantitative claims within the tested scope.

**Skeptical** – Different architectures may break modelling assumptions; real‑world profiling environments (e.g. noisy multi‑tenant clouds) could make uncertainty modelling less stable or actionable.
## Evaluation
*How is the solution evaluated, and is the evaluation convincing? Are appropriate metrics used, and does it align with original goals? Is the solution's cost justified? Is the evaluation comprehensive, covering all aspects of the solution? Are any measurements missing? Are weak points, assumptions, and limitations explored? Are complete results presented, avoiding biases and benchmarking crimes? Is the comparison against competing approaches fair? Is sufficient information provided to assess results? Is an appropriate baseline shown, and is important information not hidden by relative numbers?*

Benchmarks plus three applied scenarios (cloud orchestration, performance debugging, intrusion detection). Measures:

- Accuracy improvements over baseline tools.
- Added profiling overhead.
- Impact on downstream decision quality. Finds consistent gains in accuracy with low (reported) runtime cost.

**Pro** – Combination of synthetic and applied workloads covers both controlled and messy environments, implying some generality.

**Skeptical** – Case studies are self‑selected; results may reflect “friendly” workloads where benefits are most visible. Overhead claims could shift under sustained, large‑scale, heterogeneous deployment.
## Presentation
*Is the paper well-structured and clear? Are the problem and solution clearly presented with enough detail? Are high-level design choices explained first? Are concepts introduced systematically? Is the reader not overwhelmed with details? Is the presentation of evaluation results sensible? Are there no issues with grammar, spelling, or sloppiness?*

Classic OSDI structure: motivation → gap analysis → architecture → kernel/API design → evaluation. Frames Tintin as both a novel measurement methodology and a new OS abstraction.

**Pro** – Layered exposition makes it accessible to both kernel hackers and application‑level performance engineers; strengthens the sense of a complete, integrated system.

**Skeptical** – Strong narrative flow and polished diagrams can mask complexity of implementation; integration and maintenance costs may be under‑emphasised.