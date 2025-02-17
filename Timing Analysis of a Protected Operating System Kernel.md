[Paper](https://trustworthy.systems/publications/nicta_full_text/4863.pdf)
Calculating the [[WCET]] is done with an [[seL4]] binary compiled with GCC, `-O2` optimisation and `-fwhole-program` flag.
- aggressive inlining means the compiled code doesn't correlate well with the C source
- [[Chronos]] apparently has a feature to assist with this correlation

Absence of function pointers means a [[Control Flow Graph]] of seL4 can be extracted without user guidance. We should be able to use [[Heptane]] with a [[RISC-V]] build, then, possibly with modification.

But... the paper used [[symbolic execution]]. How does Heptane do it?
- symbolic execution is required because of evaluation of pointer arithmetic on R-W memory such as return addresses on the stack. 



