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
    
