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

# Segmentation
- Segmentation: The separation of the physical address space into code, heap, and stack segments within the processor space. All code of all processes are grouped together, all stacks are grouped together, and all heap are together.
- x86-64 does not have segmentation, and uses memory paging for memory protection instead
- Reduces the amount of unusable fragments of memory inside each processor allocation
- Enables memory protection
  - An address starts with 1 or 2 bits of segment information (code, stack, or heap)
  - By adding the offset address to the starting address and comparing it to the end address, out-of-bounds addresses can be detected
  - Protection bits indicate whether a program can read, execute code, and/or write data in specific memory ranges

# Free-Space Management
- External fragmentation: Having so much unused and unusable small bits of free space that there is little usable free space left
- Internal fragmentation: Lots of unused space inside of allocated segments
- Free list
  - A list that keeps track of free spaces in memory
  - Example: `head -> addr: 0, len: 10 -> addr: 20, len 10 -> NULL`
  - When a chunk of memory is freed, the surrounding free space must be coalesced so that the free space nodes can be combined
- Allocators track chunk sizes in a header block so that the correct amount of bytes is freed when `free()` is called
- Allocation strategies
  - Best fit
    - Allocate in the smallest possible chunk of free space that the requested size can fit
    - Must search the entire free list
  - Worst fit
    - Allocate in the largest chunk
  - First fit
    - Allocates in the first chunk that is big enough
    - Fastest way to find a usable chunk
    - May pollute the beginning of the free list with small objects
  - Next fit
    - Stores a pointer to the last allocated free space
    - Allocates in the next free space and moves the pointer
    - Spreads allocated chunks rather than polluting the beginning of the list
  - Segregated lists
    - Try to store chunks of similar sizes in one location in the address space
    - Avoids fragmentation because the same chunks fit perfectly in this location
  - Buddy allocator
    - Divides the address space into a binary tree
    - When a chunk is freed, the "buddy" of the binary node is combined if it is free
    - Faster in coalescing free space

# Paging
- Paging
  - Memory is divided into fixed-size pieces
  - Another approach to space management
  - One page is a unit of memory
  - Does not suffer from external fragmentation because of the fixed-size pages
- A fixed number of prefix bits of the virtual address contains the virtual page number
- Page table
  - A per-process data structure
  - Stores address translations between virtual pages and addresses for the corresponding pages in physical memory
  - Each entry contains the physical frame number, protection bits (read/write/execute), and a present bit which indicates if the memory is stored in RAM or on disk

# Translation-lookaside buffer
- The translation-lookaside buffer is a part of the MMU that stores commonly used page table entries
- Address space identifier (ASID)
  - Since page tables are per-process, the TLB needs a way to translate addresses based on the process
  - An ASID is like a PID that is stored with page frame numbers in the TLB
  - It indicates which process should access which entry

# Hybrid approach to paging
- Page tables can be too big with a simple implementation
- Bigger pages allow smaller page tables at the cost of more internal fragmentation
- One way to save space is to use the concept of segmentation to store the bounds for each segment
- Another is a multi-level page table
  - The page directory stores a prefix and the page table identifier with a valid bit
  - Each page table fits in a page
  - Only allocates enough page tables for used space
  - There is a trade off of necessitating a second load
  - Example virtual address:
  ```
    8 7 6 5 4 3 2 1 0
    |-a--|--b--|-c--|
    a = Page directory index
    b = Page table index
    c = Offset
  ```

# Page swapping
- Swapping pages to disk transparently enables the illusion of having a large virtual address space
- The page table entry contains a present bit to indicate if data is stored on RAM or in disk
- A page fault occurs when a read access needs to retrieve data from disk
- A page-fault handler lets the OS know that a page fault occurred and requests that the page be loaded to RAM
- Page-replacement policy
  - A policy for choosing which old pages in RAM to replace with requested pages
  - A swap daemon is activated when RAM usage is at high watermark and evicts pages until it reaches low watermark

# Threads
- Threads enable parallelization and non-blocking programs
- Threads can context switch like processes
  - Threads store context in thread control blocks
- Each thread has their own stack, but the heap is shared between all threads of a process

# Thread API
- Thread creation and joining
  ```c
  pthread_create();
  pthread_join(); // Always check the errors
  ```
- Locks
  ```c
  pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
  pthread_mutex_lock(&lock);
  pthread_mutex_unlock(&lock);
  ```


# Condition variable

```c
pthread_mutex_t mutex;
pthread_cond_t condition;

// In thread 0
pthread_mutex_lock(&mutex);
while (!condition) {
  pthread_cond_wait(&condition, &mutex); // Put in a while loop in case of of spurious wakeup
}

pthread_mutex_unlock(&mutex);

// In thread 1
pthread_mutex_lock(&mutex);

pthread_mutex_unlock(&mutex);
pthread_cond_signal(&condition); //wake up thread 1
```

- Spurious wakeup can occur when thread A is signalled to wake up, but a thread B starts before the thread A wakes up
- Producer-consumer problem
  - A shared buffer between a producer and consumer needs some synchronization mechanism to ensure the buffer is not accessed at the same time, and so the buffer doesn't become full or the consumer doesn't access an empty buffer
  - The solution is to use 3 semaphores:
    - `S`: This semaphore ensures only one consumer or producer can access the buffer
    - `E`: Indicates that the buffer is empty
    - `F`: Indicates that the buffer is full
  - Pseudocode for producer and consumer
    ```c
    void producer() {
      while (true) {
        produce()
        wait(E)
        wait(S)
        append()
        signal(S)
        signal(F)
      }
    }
    void consumer() {
      while (true) {
        wait(F)
        wait(S)
        take()
        signal(S)
        signal(E)
        use()
      }
    }
    ```

# Semaphores

```c
sem_t m;
sem_init(&m, 0, 0);
sem_wait(&m); // Waits for value to be 1


sem_post(&m); // Increments value
```

- Condition variables have a boolean state of waiting or running, and is typically used for mutually exclusive access to a variable.
- Semaphores combines a mutex and a counter, and is used for shared access to a resource.
