# Reverse Engineering Roadmap

**Goal:** Analyze compiled software without source code — understand what it does, how it works, and where it's vulnerable. Applies to malware analysis, CTF challenges, vulnerability research, and software interoperability.

---

## What Is Reverse Engineering?

Compiled software is source code that has been transformed:

```
Source Code (C/C++/Rust)
      ↓  [Compiler]
Assembly (architecture-specific mnemonics)
      ↓  [Assembler]
Machine Code (bytes the CPU executes)
      ↓  [Stored on disk]
Binary / Executable (ELF on Linux, PE on Windows, Mach-O on macOS)
```

Reverse engineering goes the other direction:
- **Disassembly**: machine code → assembly (accurate, always possible)
- **Decompilation**: machine code → pseudo-C (easier to read, sometimes wrong)
- **Debugging**: running the binary under controlled observation

---

## Phase 0 — Prerequisites

### x86-64 Assembly Basics

You do not need to write assembly. You need to *read* it.

```nasm
; x86-64 calling convention (System V AMD64 ABI — Linux)
; Function arguments: rdi, rsi, rdx, rcx, r8, r9 (then stack)
; Return value: rax

; mov — copy a value
mov rax, 1          ; rax = 1
mov rax, [rbp-8]    ; rax = value at memory address (rbp-8)
mov [rbp-8], rax    ; store rax's value at address (rbp-8)

; Arithmetic
add rax, rbx        ; rax = rax + rbx
sub rax, 5          ; rax = rax - 5
imul rax, rbx       ; rax = rax * rbx
xor rax, rax        ; rax = 0 (fast zero idiom)

; Comparisons and jumps
cmp rax, rbx        ; set flags based on rax - rbx (doesn't store result)
je  label           ; jump if equal (ZF=1)
jne label           ; jump if not equal
jl  label           ; jump if less (signed)
jg  label           ; jump if greater (signed)
jmp label           ; unconditional jump

; Stack operations
push rax            ; decrement rsp by 8, store rax at [rsp]
pop  rbx            ; load [rsp] into rbx, increment rsp by 8
call function       ; push return address, jump to function
ret                 ; pop return address, jump to it

; Function prologue/epilogue (standard pattern)
push rbp            ; save caller's base pointer
mov  rbp, rsp       ; establish new stack frame
sub  rsp, 32        ; allocate local variable space
; ... function body ...
mov  rsp, rbp       ; restore stack pointer
pop  rbp            ; restore caller's base pointer
ret
```

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [x86-64 Assembly — CS:APP Chapter 3](http://csapp.cs.cmu.edu/) | Book chapter | Free online |
| [OpenSecurityTraining2 — Architecture 1001](https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Arch1001_x86-64_Asm+2021_v1/about) | Course | Free |
| [Godbolt Compiler Explorer](https://godbolt.org/) | Browser tool — see C → ASM live | Free |

```c
// Use Godbolt to learn assembly from C
int add(int a, int b) {    // → movl %edi, -4(%rbp)
    return a + b;          //   movl %esi, -8(%rbp)  
}                          //   addl %ecx, %eax
                           //   retq
```

### C Programming (Read, Not Write)

Most compiled binaries were written in C or C++. You need to recognize common patterns:

```c
// These C constructs have recognizable assembly patterns
if (x > 0) { ... }           // cmp + conditional jump
for (int i=0; i<10; i++) {} // counter + cmp + jne (loop)
while (*ptr != 0) { ptr++; } // dereference + cmp + jne
malloc(size)                  // call to malloc, check rax for NULL
strcpy(dst, src)              // call + two pointer args in rdi, rsi
```

---

## Phase 1 — Static Analysis

Static analysis: examine the binary *without running it.*

### Triage Commands

```bash
# 1. Identify the file type
file binary
# → ELF 64-bit LSB executable, x86-64 (Linux)
# → PE32+ executable (Windows)
# → Mach-O 64-bit x86_64 executable (macOS)

# 2. Extract printable strings
strings binary
strings -n 8 binary           # minimum 8 chars
strings -e l binary           # Unicode (Windows PE files)

# 3. Check binary protections
checksec --file=binary        # apt install checksec

# 4. List dynamic library dependencies
ldd binary
objdump -p binary | grep NEEDED

# 5. List imported functions (what APIs does it call?)
objdump -d -M intel binary | grep "call"
nm -D binary                  # dynamic symbols
readelf -s binary             # all symbols

# 6. ELF section headers
readelf -S binary             # sections (.text, .data, .bss, .plt, .got)
readelf -l binary             # program headers (segments)
```

### Ghidra — The Primary Tool

Ghidra (NSA, free) is the best free disassembler + decompiler. Learn it deeply.

```
Setup:
1. Download: https://ghidra-sre.org/ (requires Java 17+)
2. Run: ./ghidraRun (Linux/Mac) or ghidraRun.bat (Windows)
3. New Project → Import File → Auto-analyze (accept defaults)
4. CodeBrowser opens — the main analysis interface

Key windows:
- Symbol Tree (left) → Functions, Labels, Namespaces
- Listing (center) → Disassembly view
- Decompiler (right) → Pseudo-C output
- Function Call Graph → Visualize call relationships
```

**Ghidra Workflow:**

```
1. Open the binary, run auto-analysis
2. Find main() or WinMain() in Symbol Tree → Functions
3. Click into the function → read the decompiler output
4. Rename variables as you understand them (press L to rename)
5. Add comments (press ; for end-of-line comment)
6. Follow function calls by double-clicking
7. Search strings to find interesting functions:
   Search → For String → "password" → find where it's used
```

**Useful Ghidra shortcuts:**

| Key | Action |
| :--- | :--- |
| `G` | Go to address |
| `L` | Rename variable/function |
| `;` | Add EOL comment |
| `X` | Find cross-references (who calls this?) |
| `F5` | Toggle decompiler view |
| `Ctrl+F` | Search in listing |
| `Ctrl+Shift+F` | Search in all strings |

### IDA Free

IDA Free is limited compared to IDA Pro but still useful for x86 and x86-64:

```
Download: https://hex-rays.com/ida-free/
Key advantage: more accurate analysis for certain binaries
Key limitation: no decompiler in free version (Hex-Rays decompiler is paid)
```

---

## Phase 2 — Dynamic Analysis

Dynamic analysis: run the binary and observe its behavior.

### GDB + pwndbg

**pwndbg** is an essential GDB plugin that adds context, color, and exploit-development helpers.

```bash
# Install pwndbg
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# Run GDB with pwndbg
gdb ./binary

# Core commands
(gdb) run                    # Start execution
(gdb) run arg1 arg2          # Start with arguments
(gdb) break main             # Breakpoint at main
(gdb) break *0x401234        # Breakpoint at address
(gdb) continue               # Resume to next breakpoint
(gdb) next                   # Step over (don't enter function calls)
(gdb) step                   # Step into function calls
(gdb) nexti                  # Next instruction
(gdb) stepi                  # Step into at instruction level

# Inspect state
(gdb) info registers         # All registers
(gdb) print $rax             # Single register
(gdb) x/10gx $rsp            # Examine 10 giant (8-byte) hex values at rsp
(gdb) x/s 0x402000           # Print string at address
(gdb) disassemble main       # Disassemble a function
(gdb) backtrace              # Call stack

# pwndbg-specific (runs automatically at each break)
# → Shows registers, disassembly at current IP, stack — all at once
(gdb) heap                   # Heap chunk visualization
(gdb) vmmap                  # Virtual memory map
(gdb) cyclic 100             # Generate a De Bruijn sequence (offset finding)
(gdb) cyclic -l $rbp         # Find offset of that value in the sequence
```

### strace / ltrace

```bash
# System call trace — what syscalls does the binary make?
strace ./binary
strace -e trace=open,read,write,execve ./binary  # Filter specific calls
strace -o trace.log ./binary                     # Save to file

# Library call trace — what C library functions?
ltrace ./binary
ltrace -e strcmp ./binary     # Only track strcmp calls (password comparison!)
```

---

## Phase 3 — Crackmes and CTF RE Challenges

Practice on designed challenges before real targets.

### Crackme Methodology

A "crackme" is a binary that requires a valid password, serial, or keygen.

```
Step 1: strings — look for hardcoded passwords
strings crackme | grep -v "^..$"  # Filter very short strings

Step 2: ltrace — watch for strcmp/strncmp with your input
echo "test" | ltrace ./crackme 2>&1 | grep -i strcmp
# If strcmp("test", "s3cr3t") appears — you found the password

Step 3: Static analysis — find the comparison in Ghidra
Search for string "Wrong" or "Incorrect" → find function → trace back to input check

Step 4: Dynamic — use GDB to watch the comparison
# Set breakpoint on strcmp / strncmp
break strcmp
run <<< "AAAA"
# When it hits: x/s $rsi  (second argument = the expected password)
```

### CTF RE Challenges

| Platform | Level | Type |
| :--- | :--- | :--- |
| [Crackmes.one](https://crackmes.one/) | All | Crackmes/keygens |
| [picoCTF Reversing](https://play.picoctf.org/) | Beginner | CTF-style RE |
| [HackTheBox RE Challenges](https://app.hackthebox.com/challenges) | Intermediate | CTF-style |
| [rev.ing](https://github.com/revng/revng) | Mixed | RE collection |

---

## Phase 4 — Anti-Analysis Techniques

Real malware and protected software use these to slow analysis.

### Anti-Debugging

```c
// 1. IsDebuggerPresent (Windows)
if (IsDebuggerPresent()) { exit(0); }

// 2. Timing check — debuggers slow execution
#include <time.h>
clock_t start = clock();
// ... do something ...
if (clock() - start > 100) { exit(0); }  // Too slow = debugger

// 3. ptrace self-check (Linux)
if (ptrace(PTRACE_TRACEME, 0, 0, 0) == -1) { exit(0); }
// Only one tracer at a time — if already traced, ptrace fails
```

**Bypass in GDB:**
```bash
# Patch the check
(gdb) set {unsigned char}0x401234 = 0x90  # NOP the check

# Or: override the function
(gdb) set $al = 0   # Make IsDebuggerPresent return 0
```

### Obfuscation

- **String encryption**: strings are XOR'd or AES'd, decrypted at runtime → set breakpoint after decryption routine
- **Control flow obfuscation**: fake jumps, dead code, opaque predicates → trace execution with debugger
- **Packing**: binary is compressed/encrypted, decompressed at runtime → dump memory after unpacking

```bash
# Detect packing via entropy
# Packed sections have entropy close to 8.0 (random-looking data)
python3 -c "
import pefile, math
pe = pefile.PE('packed.exe')
for s in pe.sections:
    data = s.get_data()
    freq = [data.count(bytes([i])) for i in range(256)]
    e = -sum((f/len(data))*math.log2(f/len(data)) for f in freq if f)
    print(f'{s.Name.decode().strip()}: entropy={e:.2f}')
"
# > 7.2 strongly suggests packing/encryption

# Unpack via memory dump
# Run the binary in sandbox, wait for it to unpack itself into memory
# Then dump the unpacked PE from memory using Process Hacker or pe-sieve
```

---

## Tools Quick Reference

| Tool | Platform | Purpose | Cost |
| :--- | :--- | :--- | :--- |
| [Ghidra](https://ghidra-sre.org/) | All | Disassembler + decompiler | Free |
| [IDA Free](https://hex-rays.com/ida-free/) | All | Disassembler (limited) | Free |
| [Binary Ninja](https://binary.ninja/) | All | Disassembler + decompiler | Paid |
| [x64dbg](https://x64dbg.com/) | Windows | Debugger | Free |
| [GDB + pwndbg](https://github.com/pwndbg/pwndbg) | Linux | Debugger + exploit dev | Free |
| [Cutter](https://cutter.re/) | All | Radare2 GUI | Free |
| [Detect-It-Easy](https://github.com/horsicq/Detect-It-Easy) | All | File type + packer detection | Free |
| [pe-bear](https://github.com/hasherezade/pe-bear) | Windows | PE format explorer | Free |
| [pwntools](https://docs.pwntools.com/) | Linux | Python exploit library | Free |

---

## Learning Path (Structured)

1. **Week 1–2:** x86-64 assembly — [OpenSecurityTraining2 Arch1001](https://p.ost2.fyi/)
2. **Week 3–4:** Ghidra basics — solve 5 crackmes on crackmes.one
3. **Week 5–6:** GDB/pwndbg — [pwn.college Debugging Refresher](https://pwn.college/)
4. **Week 7–8:** CTF RE challenges — picoCTF reversing category
5. **Month 3+:** [pwn.college RE track](https://pwn.college/program-security/reverse-engineering/) — structured curriculum

---

## References

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Reverse Engineering for Beginners (book)](https://beginners.re/) | Book — huge, free | Free |
| [OpenSecurityTraining2](https://p.ost2.fyi/) | RE courses | Free |
| [pwn.college](https://pwn.college/) | RE + pwn curriculum | Free |
| [Ghidra Official Docs](https://ghidra-sre.org/) | Documentation | Free |
| [Hacking: Art of Exploitation](https://nostarch.com/hacking2.htm) | Book — C + RE + pwn | Paid |
