# Operating systems
- Virtualization
  - Conversion of a physical resource into a virtual form
  - The OS is a resource manager
  - Virtualization of CPU: Processes allow many programs to run, despite a limited number of CPUs
  - Virtualization of memory: Each process is assigned a virtual address space that maps to a physical space
- Concurrency
  - Multi-threadding allows many threads to run at the same time
- Persistence
  - System memory in DRAM is volatile and is erased when power is lost
  - Data in hard drives or SSDs are persistent
- System calls
  - The difference between routine procedure calls and system calls is that system calls change privilege levels
  - The user mode is restricted, while kernel mode grants more privileges
  - A trap transfers control to a trap handler which raises the privilege level
  - Return-from-trap reverts back to user mode

# Processes
- A process is a running program
- The OS virtualizes the CPU so that CPU time can be shared among running processes
- Policy: A ruleset for making decisions
- Scheduling policy: A policy for deciding which process gets to run on the CPU in a given moment
- Process API
  - Create
    - Allocates resources and starts a process
    - Processes are loaded in a lazy fasion, meaning programs are loaded when needed
    - Memory space is allocated for the process
    - 3 file descriptors are opened for a process: standard in, out, error
  - Destroy: Stops and deallocates a process
  - Wait: Wait for a process to stop
  - Miscellaneous control: Other controls, such as suspending a process
  - Status: How long a process has been running, what state it is in, etc.
- Process states
  - Running: Executing instructions
  - Ready: A process is ready to run but it is on hold
  - Blocked: The process is waiting for an event, such as file I/O
- The list of processes are contained in a process list
  - Each entry contains a process control block which has the context for each process

# UNIX process API
- `fork()`
  - Runs a copy of the current program
  - All registers and memory are copied to clone program state (or a virtual space is set aside in the case of copy-on-write)
  - Starts from where `fork()` was called
  - In the parent side, `fork()` returns the child pid
  - In the child, `fork()` returns 0
- `wait()`
  - Blocks until forked process of given pid ends
  - If NULL is passed, it waits for all child processes to end
- `exec()`
  - Runs the given program to replace the current one
  - The memory is replaced for that of the new program
  - PID does not change
  - The technique is known as overlaying
  - Fork-exec: When a forked process starts a different program
- `kill()`: Used to stop a process
- `signal()`: Used to catch a signal

# Limited direct execution
- Time sharing is when CPU usage is shared between processes by time slices
- Performing restricted operations
  - At boot time, a trap table is created to map exceptions to trap handlers
  - When a program wants to perform controlled operations such as I/O, a system call is made
  - A trap instruction jumps to the kernel instructions and switches from user mode to kernel mode
  - All register state is saved to a per-process kernel stack
  - When the system call is finished, a return-from-trap instruction is called
  - Register state is restored
- The big problem in direct execution of programs is how to share the CPU and how the kernel can take back control
  - If the program is running without giving back control, how is the kernel able to regain control?
  - A program passes control back to the kernel in system calls
  - A timer interrupt periodically calls kernel code so that it can run a scheduler if needed
- In a context switch, the hardware saves the registers to the kernel stack. The kernel switches the registers in the stack to those of the new process. It returns from trap, and the hardware restores the switched registers.

# Scheduling
- A scheduling metric can be used to quantify a scheduling policy
- Turnaround time: Time when the job arrived in the system - time when the job was completed
- Fairness
  - How quickly a job can be started
  - A round-robin policy is fair
- Executing a job that takes the longest holds up all other jobs and decreases turnaround times
- Preemptive scheduler
  - A system that can pause processes to switch to other processes
  - It makes sense to preempt a long-running job to finish shorter jobs
- Response time
  - The period of time between each time-slice of a 
  - More job slices in a time period means the process feels responsive to the user
- Overlap: When a process is blocked, other processes can be run
- Multi-level feedback queue
  - Optimizes turnaround time while making a system responsive
  - The rules of a MLFQ
    1. If Priority(A) > Priority(B), A runs
    2. If Priority(A) = Priority(B), A & B run in a round-robin fashion
    3. When a job enters the system, it is placed at the highest priority
      - Ensures that short jobs are completed quickly
    4. Once a job uses up a time allotment, its priority is reduced
      - Allows newer jobs to get a higher priority
    5. After some time period, move all jobs to the highest priority
      - Avoids starving the lowest priority jobs of CPU time, so that they can at least make a litle progress even if there are higher priority jobs

# Memory
- Processes need isolation of memory through virtual address spaces

# Heap memory API
- `malloc()`
  - Allocates memory on heap
  - Parameter is number of bytes to allocate
  - Returns a `void*` which must be casted
  - `malloc(strlen(s) + 1))` allocates memory that can store a copy of `s`
- `free()` deallocates memory
- Common errors: Forgetting to allocate, not allocating enough memory, forgetting to initialize memory, forgetting to free memory, dangling pointer, double free, trying to free a pointer that is not heap memory
- `malloc()` is not a system call. It is a library call which uses `brk` to change the location of the end of the heap
- `mmap()` maps a file to memory, and can also create an anonymous mapping which is initialized to zero
- `calloc()` allocates and zeros memory
- `realloc()` increases the size of a block of memory, and gets moved to a new address if necessary
