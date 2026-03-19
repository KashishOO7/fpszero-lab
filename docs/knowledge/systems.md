# Systems Architecture

> Every security vulnerability is ultimately a violation of some architectural assumption. Understand the architecture first.

---

## The Stack Model

Digital systems are a stack of abstractions. Each layer hides complexity from the layer above it.

```
┌─────────────────────────────────┐
│  Application (your code)        │  ← What users interact with
├─────────────────────────────────┤
│  Runtime / Libraries / Stdlib   │  ← Language runtimes, glibc, etc.
├─────────────────────────────────┤
│  Operating System (kernel)      │  ← Syscalls, process mgmt, memory
├─────────────────────────────────┤
│  Hardware Abstraction Layer     │  ← Drivers
├─────────────────────────────────┤
│  CPU / RAM / Storage / Network  │  ← Physical reality
└─────────────────────────────────┘
```

Security flaws exist at every layer:

| Layer | Example Flaw |
| :--- | :--- |
| Application | SQL injection, XSS, business logic bugs |
| Runtime/Libraries | Log4Shell (Java logging library RCE) |
| OS / Kernel | DirtyPipe (Linux kernel privilege escalation) |
| Firmware / HAL | Spectre/Meltdown (CPU microarchitecture) |
| Hardware | Cold boot attacks, hardware keyloggers |

---

## CPU Fundamentals

### Registers

Registers are tiny, ultra-fast memory locations inside the CPU. Understanding them is essential for binary exploitation.

**x86-64 General Purpose Registers:**

| Register | Full Name | Purpose |
| :--- | :--- | :--- |
| `rax` | Accumulator | Return values from functions |
| `rbx` | Base | General purpose |
| `rcx` | Counter | Loop counters, 4th arg |
| `rdx` | Data | I/O, 3rd function argument |
| `rsp` | Stack Pointer | Points to top of stack |
| `rbp` | Base Pointer | Points to base of current stack frame |
| `rip` | Instruction Pointer | Address of next instruction to execute |
| `rdi` | Destination Index | 1st function argument |
| `rsi` | Source Index | 2nd function argument |

### The Call Stack

When a function is called, the CPU creates a **stack frame**:

```
High memory
┌──────────────┐
│  main() args │
├──────────────┤
│  ret address │ ← Where to return after function finishes
├──────────────┤  ← rbp (saved base pointer)
│  saved rbp   │
├──────────────┤
│  local vars  │ ← Buffer overflows overwrite from here upward
│     ...      │
├──────────────┤  ← rsp (current stack top)
│  ...         │
Low memory
```

A stack buffer overflow overwrites local variables, then the saved rbp, then the return address — changing where the CPU jumps after the function returns.

### Memory Protection Mechanisms

Modern systems implement multiple defenses:

| Protection | Mechanism | What It Prevents |
| :--- | :--- | :--- |
| **ASLR** | Randomizes memory layout at each execution | Hardcodes addresses in exploits |
| **NX/DEP** | Marks stack/heap as non-executable | Shellcode execution on stack |
| **Stack Canaries** | Places a random value before saved rbp; checks before return | Stack buffer overflows |
| **PIE** | Position Independent Executable | Predictable code addresses |
| **RELRO** | Makes GOT read-only after linking | GOT overwrite attacks |

```bash
# Check what protections a binary has
checksec --file=./binary

# Example output:
# RELRO:    Full RELRO
# STACK CANARY: Canary found
# NX:       NX enabled
# PIE:      PIE enabled
# ASLR:     Check /proc/sys/kernel/randomize_va_space → 2 = full ASLR
```

---

## Memory Layout

### Virtual Memory

Every process sees its own virtual address space (64-bit Linux, simplified):

```
0xFFFFFFFFFFFFFFFF  ← Kernel space (not user accessible)
        ↑
   Stack (grows ↓)      ← Local variables, return addresses
        |
   [unmapped gap]
        |
   Heap (grows ↑)       ← malloc()'d memory
        |
   BSS segment          ← Uninitialized global variables
   Data segment         ← Initialized global variables
   Text segment         ← Executable code (read-only)
0x0000000000000000
```

### Heap Internals

`malloc()` manages heap memory through **chunks** with metadata headers. Heap exploitation techniques (use-after-free, double-free, heap overflow) manipulate this metadata.

---

## Process & Permission Model (Linux)

### Users and Permissions

```bash
# Every process runs as a UID (user ID) and GID (group ID)
# File permissions: rwxrwxrwx → owner/group/other

ls -la /etc/shadow
# -rw-r----- 1 root shadow 1234 Jan 1 00:00 /etc/shadow
# Owner: root (read/write), Group: shadow (read), Other: none

# SUID bit — dangerous when misconfigured
# Lets a file run with the owner's permissions, not caller's
find / -perm -u=s -type f 2>/dev/null  # Find all SUID binaries
```

### Syscalls

Programs interact with the kernel through system calls:

```c
// In C: write(1, "hello", 5);
// This becomes (x86-64 Linux):
// rax = 1  (syscall number for write)
// rdi = 1  (file descriptor: stdout)
// rsi = pointer to "hello"
// rdx = 5  (number of bytes)
// syscall
```

Understanding syscalls is essential for: malware analysis (what is this process actually doing?), seccomp sandboxing (restrict what syscalls a process can make), and system call tracing (`strace`).

---

## Networking Model

See the dedicated [Networking page →](systems/networking.md)

---

## Further Reading

| Resource | Topic |
| :--- | :--- |
| [Computer Systems: A Programmer's Perspective (CS:APP)](http://csapp.cs.cmu.edu/) | Hardware/OS/architecture deep dive |
| [Operating Systems: Three Easy Pieces](https://ostep.org/) | OS internals, free book |
| [x86-64 Linux ABI](https://refspecs.linuxbase.org/elf/x86_64-abi-0.99.pdf) | Calling conventions |
| [GDB Cheatsheet](https://darkdust.net/files/GDB%20Cheat%20Sheet.pdf) | Debugging reference |
