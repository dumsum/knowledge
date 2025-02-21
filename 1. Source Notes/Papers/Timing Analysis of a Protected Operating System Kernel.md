[Paper](https://trustworthy.systems/publications/nicta_full_text/4863.pdf)

Calculating the [[Worst Case Execution Time|WCET]] is done with an seL4 binary compiled with GCC, `-O2` optimisation and `-fwhole-program` flag.
- aggressive inlining means the compiled code doesn't correlate well with the C source
- [[Chronos]] apparently has a feature to assist with this correlation

Absence of function pointers means a [[Control Flow Graph|CFG]] of seL4 can be extracted without user guidance.

But... the paper used [[symbolic execution]].
- this is required because of evaluation of pointer arithmetic on R-W memory such as return addresses on the stack. 