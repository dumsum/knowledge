
As at LionsOS release: 0.4.0 (?)

## Abstract

This document describes the design of a portability‑oriented POSIX library for LionsOS, including the structure and interfaces of the provided implementation, and presents the current scope and demonstrations of functionality. The library adapts blocking POSIX I/O semantics to LionsOS components that natively interact with the sDDF asynchronously. 

The document:
- is explicit about assumptions on the LionsOS system model and the constraints it imposes on processes, threading, and linking;
- provides context for implementing LionsOS ports of software which expects a conventional libc;
- assumes familiarity with the seL4 microkernel, Microkit, cooperative multithreading, the seL4 Device Driver Framework (sDDF) and the design principles of a LionsOS-based system more broadly; and
- should be read in conjunction with the [sDDF Design Document](https://trustworthy.systems/projects/drivers/sddf-design-latest.pdf) and the [LionsOS website](https://lionsos.org/).

## 1 Aims of the LionsOS POSIX Library

The primary aim is portability: the library exposes a sufficient POSIX subset to enable relatively straight-forward porting of applications originally written against a conventional libc. Applications that are performance critical or employ asynchronous operations are expected to use native LionsOS components and/or interface with the sDDF directly.

## 2 System Model and Architecture

### 2.1 Component interfaces

There is no separate POSIX "server" component. Each LionsOS component that requires POSIX functionality statically links its own copy of the library. Additionally, components must also link a cooperative multithreading library (for blocking) and, if sockets are used, a TCPIP stack. LionsOS provides bindings for `libmicrokitco` and `lwIP` for this purpose, but users of the library are free to provide their own implementations.

### 2.2 libc and Library interface protocol

The library is based on a fork of `musllibc`. This fork replaces architecture-specific inline assembly syscall traps (like `svc 0` on AArch64) with a generic function call mechanism. Instead of invoking syscalls directly with assembly, `musl`’s internal `__syscallN` functions now call a dispatcher function via `__sysinfo`.

In upstream `musl`, `__sysinfo` is normally a hook for vDSO (Virtual Dynamic Shared Object) support on Linux systems, letting `musl` call kernel-provided user-space functions without a full syscall. In LionsOS, this has been repurposed as a _general_ syscall dispatcher.

In other words, all libc syscall invocations are routed through the LionsOS POSIX syscall handler. For example,

```c
#define CALL_SYSINFO(n, ...) ((long(*)(long,...))__sysinfo)(n, ##__VA_ARGS__)

static inline long __syscall3(long n, long a1, long a2, long a3) {
    return CALL_SYSINFO(n, a1, a2, a3);
}
```

#### 2.2.1 Syscall Dispatch Mechanism

The syscall dispatcher is a variadic function assigned to `__sysinfo`. The first argument is the `musl` syscall number. Remaining arguments are captured in a `va_list` to be passed to specific syscall handlers. These are called via a jump table indexed by the syscall number. The table is sized to the maximum syscall number known to musl for the target platform: any unimplemented syscalls are left as `NULL` and attempts to invoke these return `ENOSYS`.

Return convention is: non‑negative value for success, negative `errno` for failure. `musl`’s syscall infrastructure maps negative `errno` to `errno` and returns `-1` to the caller.

### 2.3 Operation

#### 2.3.1 Blocking

The recommended operating mechanism is to employ two cothreads. Blocking POSIX calls execute as part of the "application code" in the component’s "main" cothread, waiting on a Microkit channel or `libmicrokitco` semaphore. The "event" cothread receives async completions or timer interrupts (via the Microkit `notified()` entrypoint) and wakes the main cothread as appropriate.

#### 2.3.2 Backend

For file I/O the backend is a file system server component conforming to the LionsOS file system protocol, reachable via a Microkit channel and command/completion queues. Currently, LionsOS supports FAT and NFS. For sockets the backend is `lwIP`'s  _raw API_, driven by periodic timer notifications.

### 2.4 File descriptor management

File descriptors are integers indexing a per-component FD table. Each FD references a control structure for either a file or socket, with a vtable of operations such as `read`, `write`, `close`, etc. FDs store their path to support `*at` operations.

## 3 Implemented Features and Semantics

### 3.1 Dynamic Memory

`musl` is built with `malloc=oldmalloc`. `brk` and `mmap` are implemented over a fixed, statically allocated region inside the component.

- `brk` returns 0 on failure.
- `mmap` fails with `ENOMEM` when the region is exhausted.
- There is no dynamic expansion, e.g. via a memory service.

### 3.2 Filesystem and Path Semantics

Currently a single filesystem is supported and "mounted" at `/`. Path resolution is rudimentary and implemented within the library. FDs store their path to enable `openat`, `mkdirat`, and related calls.

- Supported metadata calls: `stat`, `fstat`, `fstatat`.
- Current working directory is unimplemented: `AT_FDCWD` is treated as root for the purposes of `*at` calls. Note that if the call is made with an FD corresponding to an opened directory, paths can be correctly resolved.

### 3.3 Sockets and Networking

Sockets are implemented via `lwIP`’s _raw API_, linked per component. The event cothread drives `lwIP` with periodic timer notifications from an sDDF timer driver. On each tick, `lwIP`'s periodic functions are invoked. Copying is handled below the library by designated sDDF copier components.

Only core socket operations are supported. Features like `setsockopt`, `getsockopt`, and `ioctl` are not implemented. See **Appendix A.2** for a complete support table.

Only TCP connections are supported for now.

### 3.4 General I/O Semantics

#### 3.4.1 Blocking and non‑blocking

Non‑blocking mode is not supported; `O_NONBLOCK` is ignored. Calls return after transferring some number of bytes or an error. Partial reads and writes return the actual count.

#### 3.4.2 Lifetime and close

`close()` uses reference counting on socket control structures (relevant for `dup`). FD numbers can be reused after a successful closure. Failure paths during teardown are not supported in this release.

#### 3.4.3 Duplication and flags

A minimal `dup3` is available, simply copying FDs but not sharing file offsets. Reference counting is used to track the associated open file/socket for closing. `fcntl` supports only `F_SETFD` and `F_GETFL`. Flags are stored per FD.

## 4 Scope and Limitations

The library is intentionally narrow, providing the minimal functionality required to support rudimentary I/O. It assumes a static component model. Ported applications that rely on processes, signals, or threads will require adaptation to native LionsOS protocols.

### 4.1 Unsupported POSIX Features

Non‑goals for this release include performance optimisation and verification. Additionally, most functionality beyond rudimentary file or socket I/O is explicitly not supported. The below list is _non-exhaustive_:

- Process Management: `fork` and `exec` are not supported.
- Threading: No support for pthreads.
- Signals: Not implemented; `EINTR` is not a possible error.
- Readiness: `poll`, `select`, `epoll` are not implemented.

### 4.2 Resource Limits

Currently the following limits are imposed:

- Maximum file descriptors: 128 per component.
- Maximum path length: 128 bytes.
- No `getrlimit` or `setrlimit`. No quotas.

These constants are configurable at build time and do not represent a fundamental design constraint

### 4.3 Security

The library and its callers are untrusted. This is primarily due to the provided libc, `musl`, being unverified. Protection, however, is provided by the system design and seL4 microkernel capabilities granted to components by the seL4 Microkit, on which LionsOS is built. The library itself provides no additional security guarantees.

`getrandom()` is component‑local and insecure. It currently fills bytes via `musl`’s `rand()`. Entropy guarantees are undefined. The call should be treated as best effort only.

## 5 Interface Contract and Versioning

### 5.1 Error Handling

Errors are intended to be POSIX‑canonical end to end. The dispatcher returns negative `errno` values and `musl` sets `errno` and returns `-1` to the caller. Error handling is currently **incomplete**; some syscalls return `-1` as a placeholder. Completing this implementation is a current priority.

### 5.2 Versioning and Linking

Components are expected to rebuild and relink when the library changes.

The `__sysinfo` contract is considered internal to this libc fork. Syscall numbers follow `musl`’s numbering. There is no runtime feature discovery: users rely on source and documentation.

## 6 Evaluation

LionsOS _demonstrates_ basic I/O for files and sockets in example LionsOS systems. There are no performance targets at this stage as the goal is functional portability. Explicit testing is limited.

A LionsOS port of the `wasm-micro-runtime` (WAMR) has been provided as a new LionsOS component. WAMR has built-in POSIX-based implementations of WASI syscalls. This was chosen to demonstrate the ease of porting applications which assume a traditional libc; the required porting effort was quite small.

A compatibility table of implemented calls and known deviations is provided in Appendix A. Further testing is future work.

## Appendix A: Implemented Calls

### A.1 Files and directories

| Area             | Call                               | Status                | Notes                                      |
| ---------------- | ---------------------------------- | --------------------- | ------------------------------------------ |
| Open             | `open`, `openat`                   | Implemented           | `AT_FDCWD` treated as `/` or FD path       |
| I/O              | `read`, `readv`, `write`, `writev` | Implemented           | Blocking only; partial counts returned     |
| Close            | `close`                            | Implemented           | Reference counted teardown                 |
| Seek             | `lseek`                            | Implemented           | File pointer stored per FD                 |
| Metadata         | `stat`, `fstat`, `fstatat`         | Implemented           | Others return `ENOSYS`                     |
| Duplication      | `dup3`                             | Partially implemented | Reference counted; offset/flags not shared |
| Create Directory | `mkdirat`                          | Implemented           |                                            |
| Remove Directory | `rmdir`                            | Not implemented       | Return `ENOSYS`                            |
| Flags            | `fcntl`                            | Implemented           | Subset only: `F_SETFD`, `F_GETFL`          |

### A.2 Sockets

| Area          | Call                                 | Status                | Notes                                      |
| ------------- | ------------------------------------ | --------------------- | ------------------------------------------ |
| Creation      | `socket`                             | Implemented           | Backend is `lwIP` raw API                  |
| Bind/connect  | `bind`, `connect`                    | Implemented           | Blocking semantics                         |
| Listen/accept | `listen`, `accept`                   | Implemented           | Blocking semantics                         |
| Send/recv     | `send`, `sendto`, `recv`, `recvfrom` | Implemented           | Blocking only; partial counts returned     |
| Shutdown      | `shutdown`                           | Not implemented       | Return `ENOSYS`                            |
| Options       | `setsockopt`, `getsockopt`           | Not implemented       | Return `ENOSYS`                            |
| IOCTL         | `ioctl`                              | Not implemented       | Return `ENOSYS`                            |

Note: standard I/O such as `read` and `write` is also available for sockets when invoked with an appropriate FD.

### A.3 Memory and misc

| Area   | Call          | Status                 | Notes                                                                                |
| ------ | ------------- | ---------------------- | ------------------------------------------------------------------------------------ |
| Memory | `mmap`, `brk` | Implemented            | Fixed static region; `mmap` fails `ENOMEM` on exhaustion; `brk` returns 0 on failure |
| Random | `getrandom`   | Implemented (insecure) | Fills via `rand()`; best effort only                                                 |
