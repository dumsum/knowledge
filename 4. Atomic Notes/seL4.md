### RISC-V execution paths
`trap_entry`
- kernel execution starts here
- examines `scause` register for the cause of the trap, refer to [[The RISC-V Instruction Set Manual Volume II - Privileged Architecture|RISC-V Privileged ISA]]
	- $\texttt{scause}<0 \implies \texttt{interrupt}$
		- in other words, most significant bit set
	- $\texttt{scause} = 8 \implies \texttt{exception}$
		- environment call from U-Mode
	- $\texttt{else} \implies \texttt{handle\_syscall}$

`interrupt`
- jump to `c_handle_interrupt`

`c_handle_interrupt`
- call `handleInterruptEntry`
- noreturn call `restore_user_context`

`handleInterruptEntry`
- call `handleInterrupt`
- call `schedule`
- call `activateThread`
- call `chargeBudget`

`handleInterrupt`
- call `sendSignal`
- call `halt` 

`exception`
- jump to `c_handle_exception`

`c_handle_exception`
- call  `switchLocalFpuOwner`
- call `handleVMFaultEvent`
- call `handleUserLevelFault`
- noreturn call `restre_user_context`

`handle_syscall`
- jump to `c_handle_syscall`

`c_handle_syscall`
- noreturn call `slowpath`

`slowpath`
- call `handleSyscall`
- call `handleUnknownSyscall`
- noreturn call `restore_user_context`

`handleSyscall`
- call `chargeBudget`
- call `setThreadState`
- call `schedule`
- call `activateThread`
- call `handleInvocation`
- call `handleRecv`
- call `handleInterrupt`
	- #todo `handleInterrupt` is it a preemption point in `handleSyscall`?
- call `halt`
- return

`handleUnknownSyscall`
- call `handleFault`
- call `schedule`
- call `activateThread`
- call `chargeBudget`
- call `setThreadState`
- return

`c_handle_fastpath_call`
- noreturn call `slowpath`
- call `__clzdi2`
- call `switchLocalFpuOwner`
- S-Mode return

`c_handle_fastpath_reply_recv`
- noreturn call `slowpath`
- call `__clzdi2`
- call `switchLocalFpuOwner`
- S-Mode return

`restore_user_context`
- call `switchLocalFpuOwner`
- S-Mode return

`halt`
- #todo how does this play into execution path?