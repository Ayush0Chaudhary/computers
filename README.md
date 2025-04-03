# Computers

Computers are simple: Dont be scared, if you feel this is simple, because at its core, it is.

## Architecture
INTEL 4004 first cpu

![Screenshot from 2025-03-25 22-19-35](https://github.com/user-attachments/assets/67837c2e-2e09-4b62-a116-fd5c246ccc1b)
Federico Faggins
![Screenshot from 2025-03-25 22-20-38](https://github.com/user-attachments/assets/738db1d0-4ab1-4808-9b66-e8514d8289ba)


Instructions: its is like line of code

Machine Code: set of instructions

Using tool https://defuse.ca/online-x86-assembler.htm#disassembly

Assembly Code (For Humans):
```assembly
add eax, 512
sub eax, 200
```
Machine Code 
```binary
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
Rather than the OS deciding when to preempt programs, the programs themselves would choose to yield to the OS. They would trigger a software interrupt to say, “hey, you can let another program run now.” These explicit yields were the only way for the OS to regain control and switch to the next scheduled process.


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




### What’s a UAPI?
- UAPI (Userspace API) is a part of the Linux kernel that `exposes certain constants and structures to user-space programs`.
- Example: `BINPRM_BUF_SIZE` is defined in include/uapi/linux/binfmts.h, making its value available to user-space programs.
- Before 2012, kernel and user-space headers were mixed together. The UAPI directory was created to separate them and improve maintainability.



