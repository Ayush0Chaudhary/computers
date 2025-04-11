# How Computers Work

Computers are fundamentally simple machines that follow basic principles. If this seems straightforward - good! That means you're understanding the core concepts correctly.

## Computer Architecture Basics

### Historical Context
- **Intel 4004** (1971): The world's first commercially available microprocessor
- **Key Figure**: Federico Faggin, lead designer of the 4004
  ![Intel 4004 Chip](https://github.com/user-attachments/assets/67837c2e-2e09-4b62-a116-fd5c246ccc1b)
  ![Federico Faggin](https://github.com/user-attachments/assets/738db1d0-4ab1-4808-9b66-e8514d8289ba)

### From Code to Execution
1. **Instructions**: Fundamental operations (like single lines of code)
2. **Machine Code**: Binary instructions the CPU understands directly
3. **Assembly Code**: Human-readable representation of machine code

#### Example Conversion:
```assembly
; Assembly (Human-readable)
add eax, 512  ; Add 512 to EAX register
sub eax, 200  ; Subtract 200 from EAX register
```
```
; Machine Code (Binary)
0: 05 00 02 00 00     add  eax,0x200
5: 2d c8 00 00 00     sub  eax,0xc8
```
`eax` is a register
Other general Purpose Registers

| Register           | 32-bit | 64-bit Name | Common Use                              |
|--------------------|---------|-------------|-----------------------------------------|
| Accumulator        | EAX     | RAX         | Arithmetic, return values               |
| Base               | EBX     | RBX         | Memory addressing (pointer-like)        |
| Counter            | ECX     | RCX         | Loop counters (e.g., `for` loops)       |
| Data               | EDX     | RDX         | Extended arithmetic (e.g., 64-bit results) |
| Source Index       | ESI     | RSI         | String/memory operations (source)       |
| Destination Index  | EDI     | RDI         | String/memory operations (destination)  |
| Stack Pointer      | ESP     | RSP         | Tracks the top of the stack             |
| Base Pointer       | EBP     | RBP         | Used for function stack frames          |


#### Instruction Pointer
CPU cannot run an instruction without loading it into the RAM <br>
To store the location of instruction in RAM, we use instruction pointer <br>
![cycle](https://cpu.land/images/fetch-execute-cycle.png) <br>
https://github.com/Ayush0Chaudhary/computers/blob/main/exe-fetch.md

pointer -> instruction 1 <br>
cpu fetch instruction <br>
cpu execute the instruction <br>
pointer -> instruction 2 <br>
.... <br>
....loop <br>


##### Other Aspects:  
1. How will the instruction pointer behave?  
  a. reusable instruction needed to be used.  
  b. if else like conditional statements instruction.  

##### Registers (in detail):  
1. Registers are small storage buckets that are extremely fast for the CPU to read and write to
2. a fixed set of registers (Some in table above)
3. Some registers are directly accessible from machine code. (Not all are accessible)
4. Some special internal register can be accessed by special instruction. (For example JUMP)

#### Processors  

cpu fetch instruction, and just execute them, thats it. instruction pointer is helping them to do this.  

![IP](https://cpu.land/images/instruction-pointer.png)

Its like a person sitting in a dark room with a lamp, who is just excuting the task given to him at a particular moment.  


##### Some Questions: 
1. How does that person perform multitasking. (Our computer run browser, vscode, slack together right ?)
2. Whats stopping the browser program giving instruction to shutdown the PC
3. Whats stopping other program from using instruction of other programs? (For example, why cant i write program to do something in my browser which it has not given me API for?)


#### Kernel
1. When you boot up your computer, the instruction pointer starts at a program somewhere. That program is the kernel.
2. kernel has near-full access of RAM
3. Is in charge of running software installed on your computer (known as userland programs)
4. Example of Kernel ? (L---x)

#### 2 Rings of level of security  
1. User Mode (User Ring) [Less Access]
2. Kernel Mode (Kernal Ring) [All Sccess]
   This CPL data is stored in the Register `cs`, which is 2bit register.
   1. `00` -> Kernel Mode
   2. `11` -> User Mode
   3. `01` and `10` -> (Not used in latest systems) Used by old architecture mainly for driver custom privilage
  


### Syscall
1. If userland dont have access to I/O, how does camera app manages to save photos etc etc
   - Here comes the Syscall, this help userland get access to accessed privilages of kernel Mode
   - This access is gained with the help of mechanism call `System interrupts`
       - Software interrupts are a mechanism used by processors to accomplish these transfers. They're like a `help call` to the kernel, allowing user-level tasks to request and utilize kernel services. When a software interrupt occurs, the `processor temporarily halts the current user-level task`, `hands over control to the kernel`, and then waits for the kernel to complete the requested service before resuming the user-level task.
       - The Interrupt Vector Table (IVT) is a data structure used by operating systems to manage interrupt handling in a computer system. It's essentially a table that contains a list of addresses corresponding to specific interrupts, each address pointing to a routine (also known as an interrupt handler) that will be executed when the corresponding interrupt occurs.
       - **(OLD WAYS)** If a hardware device generates an interrupt, the processor will consult the IVT to determine the appropriate interrupt handler to execute. The handler will then manage the interrupt, perform any necessary tasks, and return control back to the user-level task that was interrupted.
       - **[EXTRA]** Userland programs can use an instruction like `INT` which tells the processor to look up the given interrupt number in the `IVT`, and it uses an instruction like IRET to tell the CPU to switch back to user mode and return the instruction pointer to where it was when the interrupt was triggered.
       - IVT is preconfigured in `Kernel Mode` and stored in `RAM` during the boot process
   - **[EXTRA]** Syscall, their implementation can very a lot, and we cannot expect the developer to understand the syscall, as they vary for different OS, and same OS diff versions. So they provide wrapper lib (libc in UNIX)


## Why is your CPU not Stuck in single loop  
1. Multi-core processor ?
### Fake parallelism  
You give a process sometime on the CPU and move onto next process when time is up.  

  
2. Who decide process time is up ?
All the computer comes with a `Timer Chips`  

3. How do they work ?
- OS: Bro Going for executing the this dude Paint App Program
- Time Chip: Ok bro, will wake you up after 1ms (Because I am Linux CFS - Completly Fair Scheduler)
- OS: ok working my process, going to the Userland
- Timer Chip: Time to move on bro
- OS: Ok let me save my current progress 
- Timer Chip: Cool!
- OS: Ok, time to move on to Slack, gotta show him his notifications.

Above process is called `preemptive multitasking`

### Timeslices (Quantums)
Duration the Timer Chip run the program.
Q. What is the problem with too small and too big timeslice ?  
ans. `Too Small`: Process switching is computationally expensive.
     `Too Big`: Not anymore multi tasking, processes are stuck  
1. how do you calculage this timeslice?
   1. `fixed timeslice round-robin`: 10ms fixed time. Not very efficient.
   2. `Target Latency`: the longest time a process should wait before running again. If OS target latency is 6ms. each process gets 1.5ms
      - if too many process, have a min limit, because timeslice will become too small.
      - Linux’s scheduler uses a target latency of 6 ms and a minimum granularity of 0.75 ms.
   3. `Completely Fair Scheduler`: Used by Linux since 2007, interesting stuff, you can read about it. https://docs.kernel.org/scheduler/sched-design-CFS.html


### History
### `Cooperative Multitasking1`  
Rather than the OS deciding when to preempt programs, the programs themselves would choose to yield to the OS. They would trigger a software interrupt to say, `“hey, you can let another program run now.”` These explicit yields were the only way for the OS to regain control and switch to the next scheduled process.


## How to run a program? 
![Progrm flow](https://cpu.land/images/linux-program-execution-process.png)

1. `execve` :  It loads a program and, if successful, replaces the current process with that program  
### `execve` Call Signature
```c
int execve(const char *filename, char *const argv[], char *const envp[]);
```

#### filename (Program Path)

This is the path to the executable you want to run.

`Example: "/bin/ls" if you want to run the ls command.`

#### argv (Arguments List)

A list of strings passed to the new program.

The last item in the list is `NULL` (this helps the system know where the list ends).

`Example: If you run ls -l /home, the argv list would be:`
```
["ls", "-l", "/home", NULL]
```
#### envp (Environment Variables List)

Another list of strings, but these are environment variables that the program can access.

Also NULL-terminated (to mark the end).

Usually written as KEY=VALUE pairs.

Example:
```
["PATH=/usr/bin", "HOME=/home/user", NULL]
```
### Define execve

Below is Macro defining the `execve`  
[link to code](https://github.com/torvalds/linux/blob/22b8cc3e78f5448b4c5df00303817a9137cd663f/fs/exec.c#L2105-L2111)
```c
SYSCALL_DEFINE3(execve,
		const char __user *, filename,
		const char __user *const __user *, argv,
		const char __user *const __user *, envp)
{
	return do_execve(getname(filename), argv, envp);
}
```

```
const char * → A string (character array).

__user → The data comes from user space (not kernel space).

const char __user *const __user * → A pointer to an array of user-space strings (like argv and envp).
```

The `getname` return the filename struct.  
It copies the string from user space to kernel space and does some usage tracking things  

[likn to code](https://github.com/torvalds/linux/blob/22b8cc3e78f5448b4c5df00303817a9137cd663f/include/linux/fs.h#L2294-L2300)
```c
struct filename {
	const char		*name;	/* pointer to actual string */
	const __user char	*uptr;	/* original userland pointer */
	int			refcnt;
	struct audit_names	*aname;
	const char		iname[];
};
```

The `do_execve` is defined below  
```c
static int do_execve(struct filename *filename,
	const char __user *const __user *__argv,
	const char __user *const __user *__envp)
{
	struct user_arg_ptr argv = { .ptr.native = __argv }; /* Clean code for Kernel */
	struct user_arg_ptr envp = { .ptr.native = __envp }; /* Clean code for Kernel */
	return do_execveat_common(AT_FDCWD, filename, argv, envp, 0);
}
```
```
AT_FDCWD -> relative to current directory
rest have same meaning
```

- do_execveat_common starts by setting up linux_binprm, which holds all the info needed to run a new process.
- It reads the first `256 bytes` of the executable to figure out what kind of file it is. Earlier it was `128 bytes`. for example `#!/usr/bin/python`
- If the file is a script, it finds the interpreter and prepares to run that instead.
- UAPI ensures some constants are exposed to user-space for better interaction with the kernel.

#### `linux_binprm`
1. Virtual Memory Setup

	- Structures like mm_struct (memory management) and vm_area_struct (virtual memory regions) are initialized.
	- This prepares the memory for the new program.
2. Argument Count (argc) and Environment Count (envc)
	- The number of command-line arguments (argc) and environment variables (envc) are counted and stored.
	- These will be passed to the new program when it starts.

3. Filename and Interpreter (filename & interp)
	- `filename`: Stores the path of the file being executed.
	- `interp`: If the program is an interpreted script (like Python), this field will point to the interpreter (e.g., /usr/bin/python instead of script.py).
	- Initially, filename == interp, but they may diverge if the script has a shebang (#!).

4. Buffer (buf) for File Detection
	- The first 256 bytes of the executable file are read into buf (size defined by BINPRM_BUF_SIZE).
	- The kernel uses this to:
		- Detect file formats (ELF, script, etc.).
		- Read shebangs (e.g., #!/usr/bin/python).
		- Decide how to execute the file.
```c
/* file -> include/linux/binfmts.h */
	char buf[BINPRM_BUF_SIZE];
```
```c
/* file -> include/uapi/linux/binfmts.h */
/* sizeof(linux_binprm->buf) */
#define BINPRM_BUF_SIZE 256
```




### What’s a UAPI?
- UAPI (Userspace API) is a part of the Linux kernel that `exposes certain constants and structures to user-space programs`.
- Example: `BINPRM_BUF_SIZE` is defined in include/uapi/linux/binfmts.h, making its value available to user-space programs.
- Before 2012, kernel and user-space headers were mixed together. The UAPI directory was created to separate them and improve maintainability.


### Step 2: Identifying the Program Type (Using binfmt Handlers)

#### 1. What Are binfmt Handlers?
- Handlers are pieces of code that understand different executable formats (e.g., ELF, shell scripts, Python scripts).
- Each handler is responsible for loading and preparing the program for execution.

- The handlers are found in files like:
	- `fs/binfmt_elf.c` → Handles ELF binaries (compiled executables).
	- `fs/binfmt_script.c` → Handles script files (#!/bin/bash or #!/usr/bin/python).
	- `fs/binfmt_flat.c` → Used for embedded systems with flat binaries.

#### How Does the Kernel Identify the Program Type?
Each handler has a function called `load_binary()`, which checks if it can handle the file.  

This function usually does the following:
- Checks for `Magic Numbers` (File Signatures)
	- Many file types start with special bytes called magic numbers.
	- Example:
		- ELF binaries start with: 0x7F 45 4C 46 (0x7F ELF).
		- Scripts usually start with #! (shebang).

- Reads the First Few Bytes (buf from linux_binprm)
	- The kernel loads the first 256 bytes of the file into a buffer (buf).
	- The handler examines these bytes to detect the format.

- Checks File Extensions (Sometimes)
	- Some formats may rely on file extensions (less common in Linux).

#### If the Handler Recognizes the Format
- It prepares the program for execution.
	- Returns a success code.
- If the Handler Does NOT Recognize the Format
	- It returns an error, and the kernel tries the next handler.

### Recursive Execution (Chained Interpretation)
- Sometimes, a file isn’t directly executable but is instead a script that requires an interpreter.

- The kernel resolves this recursively:

	- Example: Running a Python script (script.py with #!/usr/bin/python):
		- binfmt_script sees the #! and finds python.
		- binfmt_elf loads python, which is an ELF executable.
		- Execution starts from the python binary.
		- If the interpreter itself is another script, this process repeats.

Example of a recursive chain:

```sql
binfmt_script  →  binfmt_script  →  binfmt_elf
(script.py)       (wrapper.sh)        (python)
```

At the end of this chain, an actual ELF binary (e.g., Python) gets executed.


#### How does scripts work ?
```c
 // fs/binfmt_script.c
	/* Not ours to exec if we don't start with "#!". */
	if ((bprm->buf[0] != '#') || (bprm->buf[1] != '!'))
		return -ENOEXEC;
```
[code link](https://github.com/torvalds/linux/blob/22b8cc3e78f5448b4c5df00303817a9137cd663f/fs/binfmt_script.c#L40-L42)

So if it does start with `#!`, it reads the interpretor path until `newline` or `space`. 

- 2 Point to be noted my lord!
	1. More than 256 byte interpretor will be ignored and everything will fail. Earlier it was 128.
 	2. Argument Modification.
- Example
1. Let’s look at a sample execve call:
	- `execve("./script", [ "A", "B", "C" ], []);`
2. This hypothetical script file has the following shebang as its first line:
	- `#!/usr/bin/node --experimental-module`
3. The modified argv finally passed to the Node interpreter will be:
	- `[ "/usr/bin/node", "--experimental-module", "./script", "B", "C" ]`

 #### `binfmt_misc`
- binfmt_misc is a flexible way to define new executable formats without kernel modifications.
[EXTRA]
run
```cli
ls ../../proc/sys/fs/binfmt_misc/
```
this will show all the registered formats
```
(base) panda@panda-LOQ-15IRX9:~$ cat  ../../proc/sys/fs/binfmt_misc/python
enabled
interpreter /usr/bin/python3
flags: 
offset 0
magic 89505954484f4e

```
if you wanna run your script like below 
```
$ ./test.xyz
```
instead of 
```
$ python text.py
```
you can use binfmt_misc

- [extra] This binfmt_misc system is often used by Java installations, configured to detect class files by their `0xCAFEBABE` magic bytes and JAR files by their extension.

## ELF
Executable and Linkable File
- All the programs at the end become a ELF file(elf in Linux, Mach-O in MacOS, and .exe uses `Portable Executable` format in Windows)
- Linus support other format too, but ELF is most common
![ELF structure](https://cpu.land/images/elf-file-structure.png)

### Elf Header
- `Identifies the processor architecture`: It tells the system whether the binary is for x86, ARM, RISC-V, or another architecture.
- `Specifies the binary type`: It can be `Executable` → A standalone program that runs on its own or `Shared Library` → A dynamically linked library (.so file) used by other programs.
- `Defines the program’s entry point`: such as `Program header` and `section` Table

[TRY OUT YOURSELF]
```
base) panda@panda-LOQ-15IRX9:~/pr/go$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Position-Independent Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6d30
  Start of program headers:          64 (bytes into file)
  Start of section headers:          140328 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         31
  Section header string table index: 30
```

### Program Header Table
How It Works
Each entry in the Program Header Table provides:

1. Where its data is in the ELF file  
Example: "Start reading data from byte 500 in the file."

2. Where to place this data in memory (if needed)  
Example: "Load this at memory address 0x8048000."

3. How much memory it needs  
Example:  
- "It takes 4 KB in the file but needs 8 KB in memory."
- The extra 4 KB will be filled with zeroes (used for uninitialized variables → BSS segment).
4. Permissions (Flags)
	- PF_R → Readable
	- PF_W → Writable
	- PF_X → Executable
Example: "This section contains code, so set it as PF_R | PF_X."

```
(base) panda@panda-LOQ-15IRX9:~/pr/go$ readelf -l /bin/ls

Elf file type is DYN (Position-Independent Executable file)
Entry point 0x6d30
There are 13 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000000040 0x0000000000000040
                 0x00000000000002d8 0x00000000000002d8  R      0x8
  INTERP         0x0000000000000318 0x0000000000000318 0x0000000000000318
                 0x000000000000001c 0x000000000000001c  R      0x1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x00000000000036f8 0x00000000000036f8  R      0x1000
  LOAD           0x0000000000004000 0x0000000000004000 0x0000000000004000
                 0x0000000000014db1 0x0000000000014db1  R E    0x1000
  LOAD           0x0000000000019000 0x0000000000019000 0x0000000000019000
                 0x00000000000071b8 0x00000000000071b8  R      0x1000
  LOAD           0x0000000000020f30 0x0000000000021f30 0x0000000000021f30
                 0x0000000000001348 0x00000000000025e8  RW     0x1000
  DYNAMIC        0x0000000000021a38 0x0000000000022a38 0x0000000000022a38
                 0x0000000000000200 0x0000000000000200  RW     0x8
  NOTE           0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  NOTE           0x0000000000000368 0x0000000000000368 0x0000000000000368
                 0x0000000000000044 0x0000000000000044  R      0x4
  GNU_PROPERTY   0x0000000000000338 0x0000000000000338 0x0000000000000338
                 0x0000000000000030 0x0000000000000030  R      0x8
  GNU_EH_FRAME   0x000000000001e170 0x000000000001e170 0x000000000001e170
                 0x00000000000005ec 0x00000000000005ec  R      0x4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     0x10
  GNU_RELRO      0x0000000000020f30 0x0000000000021f30 0x0000000000021f30
                 0x00000000000010d0 0x00000000000010d0  R      0x1

 Section to Segment mapping:
  Segment Sections...
   00     
   01     .interp 
   02     .interp .note.gnu.property .note.gnu.build-id .note.ABI-tag .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt 
   03     .init .plt .plt.got .plt.sec .text .fini 
   04     .rodata .eh_frame_hdr .eh_frame 
   05     .init_array .fini_array .data.rel.ro .dynamic .got .data .bss 
   06     .dynamic 
   07     .note.gnu.property 
   08     .note.gnu.build-id .note.ABI-tag 
   09     .note.gnu.property 
   10     .eh_frame_hdr 
   11     
   12     .init_array .fini_array .data.rel.ro .dynamic .got 
```

#### Section Table

 Detailed Explanation of Each Section
1. `.text` (Machine Code)
Stores the executable instructions of the program.

Marked as read-only and executable (SHF_ALLOC, SHF_EXECINSTR).

When the CPU executes a program, it starts here!

2. `.data` (Initialized Global Variables)
Holds initialized global/static variables (e.g., int x = 5;).

Read/write section (SHF_ALLOC, SHF_WRITE).

Loaded into memory as part of the LOAD segment.

3. `.bss` (Uninitialized Variables)
Holds uninitialized global/static variables (e.g., int y;).

Doesn't take up space in the ELF file (SHT_NOBITS).

Memory is allocated at runtime and initialized to zero.

SHF_ALLOC, SHF_WRITE flags (allocated and writable).

4. `.rodata` (Read-Only Data)
Stores constants and string literals (printf("Hello")).

Read-only (SHF_ALLOC).

Prevents accidental modification.

5. `.shstrtab` (Section Header String Table)
Stores names of sections (e.g., .text, .data).

Each section header entry contains an offset into .shstrtab.

Makes parsing easier (fixed-size numbers instead of variable strings).

6. `.symtab` (Symbol Table)
Stores function and variable names.

Used by linkers and debuggers.

Stripped in production binaries for security and size.

7. `.strtab` (String Table)
Stores string names for symbols in .symtab.

Keeps .symtab entries compact (by using offsets).

8. .dynamic (Dynamic Linking Info)
Stores information for dynamically linked libraries (libc.so, etc.).

Used by dynamic loaders like ld.so.

`OFFSET` -> How many bytes into the binary file is a particular peice of code is?
- This means the .text section starts at byte 2432 (decimal) inside the ELF file.

```
(base) panda@panda-LOQ-15IRX9:~/pr/go$ readelf -S /bin/ls
There are 31 section headers, starting at offset 0x22428:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.pr[...] NOTE             0000000000000338  00000338
       0000000000000030  0000000000000000   A       0     0     8
  [ 3] .note.gnu.bu[...] NOTE             0000000000000368  00000368
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000038c  0000038c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003b0  000003b0
       0000000000000050  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           0000000000000400  00000400
       0000000000000c00  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000001000  00001000
       0000000000000614  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           0000000000001614  00001614
       0000000000000100  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          0000000000001718  00001718
       00000000000000f0  0000000000000000   A       7     2     8
  [10] .rela.dyn         RELA             0000000000001808  00001808
       0000000000001530  0000000000000018   A       6     0     8
  [11] .rela.plt         RELA             0000000000002d38  00002d38
       00000000000009c0  0000000000000018  AI       6    25     8
  [12] .init             PROGBITS         0000000000004000  00004000
       000000000000001b  0000000000000000  AX       0     0     4
  [13] .plt              PROGBITS         0000000000004020  00004020
       0000000000000690  0000000000000010  AX       0     0     16
  [14] .plt.got          PROGBITS         00000000000046b0  000046b0
       0000000000000040  0000000000000010  AX       0     0     16
  [15] .plt.sec          PROGBITS         00000000000046f0  000046f0
       0000000000000680  0000000000000010  AX       0     0     16
  [16] .text             PROGBITS         0000000000004d70  00004d70
       0000000000014032  0000000000000000  AX       0     0     16
  [17] .fini             PROGBITS         0000000000018da4  00018da4
       000000000000000d  0000000000000000  AX       0     0     4
  [18] .rodata           PROGBITS         0000000000019000  00019000
       000000000000516f  0000000000000000   A       0     0     32
  [19] .eh_frame_hdr     PROGBITS         000000000001e170  0001e170
       00000000000005ec  0000000000000000   A       0     0     4
  [20] .eh_frame         PROGBITS         000000000001e760  0001e760
       0000000000001a58  0000000000000000   A       0     0     8
  [21] .init_array       INIT_ARRAY       0000000000021f30  00020f30
       0000000000000008  0000000000000008  WA       0     0     8
  [22] .fini_array       FINI_ARRAY       0000000000021f38  00020f38
       0000000000000008  0000000000000008  WA       0     0     8
  [23] .data.rel.ro      PROGBITS         0000000000021f40  00020f40
       0000000000000af8  0000000000000000  WA       0     0     32
  [24] .dynamic          DYNAMIC          0000000000022a38  00021a38
       0000000000000200  0000000000000010  WA       7     0     8
  [25] .got              PROGBITS         0000000000022c38  00021c38
       00000000000003c8  0000000000000008  WA       0     0     8
  [26] .data             PROGBITS         0000000000023000  00022000
       0000000000000278  0000000000000000  WA       0     0     32
  [27] .bss              NOBITS           0000000000023280  00022278
       0000000000001298  0000000000000000  WA       0     0     32
  [28] .gnu_debugaltlink PROGBITS         0000000000000000  00022278
       0000000000000049  0000000000000000           0     0     1
  [29] .gnu_debuglink    PROGBITS         0000000000000000  000222c4
       0000000000000034  0000000000000000           0     0     4
  [30] .shstrtab         STRTAB           0000000000000000  000222f8
       000000000000012f  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), l (large), p (processor specific)
```

## Linking

There is 2 type of linking, static and dynamic

![static vs dynamic linking](https://cpu.land/images/static-vs-dynamic-linking.png)

1. `Dynamic Linking` -> We say we need this lib  
2. `Static Linking`->  We add the needed lib in the binary to be distributed  


#### Shared Libraries Across Operating Systems
- **Linux**: Uses `.so` (Shared Object) files, which are ELF files containing a `.dynsym` section that exports symbols for dynamic linking.
- **Windows**: Uses `.dll` (Dynamic Link Library) files, which serve the same purpose but are formatted differently.
- **macOS**: Uses `.dylib` (Dynamically Linked Library) files, following a similar approach to dynamic linking.

1. dynamic load the entire code of Library in memory
2. Which method is better for computer in general, dynamic or static ? (Static method copies a certain part of shared lib code)


#### Excecution
1. Special Library load stuf into your computer RAM. Dynamicaly linked library are loaded, and their jump pointer are created. etc etc... Whole complex process.
2. We remove all the argc, argv, environment variable from the registers.


## 2 Questions
1. Why Don’t Processes Overlap in Memory? (Virtual Memory & MMU)   
2. How Are Processes Created, like first process? (Fork, Execve, and the First Process)

Answer 1: **Memory is Fake!**  
**Virtual Memory** : This is a memory given to a program, where all they think they have entire table
**Memory Management Unit (MMU)**: Gives a actual location in RAM when CPU give virtual memory.
![diagram of how CPU and MMU works](https://cpu.land/images/virtual-memory-mmu-example.png)


1. Each process have a paging table which is basically a dictionary.
2. This helps to point to exact locatoin in the real memory, and offset is bassically same. (How ?)


![h](https://cpu.land/images/4kib-paging-address-breakdown.png)

![ff](https://cpu.land/images/process-virtual-memory-mapping.png)



```
63          47          38          29          20          11          0
┌───────────┬───────────┬───────────┬───────────┬───────────┬───────────┐
│   SIGN    │   PML4    │   PDPT    │    PD     │    PT     │  OFFSET   │
└───────────┴───────────┴───────────┴───────────┴───────────┴───────────┘
```



