# Introduction

- Running a program is simple, you're just running a set of instructions.

- The processor `fetches` an instructions form memory, `decodes` it and `executes` it then moves on to the next instruction or the program finishes
    - i.e the fetch-decode-execute cycle or `Von Neumann Model of Computing`

- While a program runs, lots of other things are happening so that the system is easy to use

- The body of software responsible for making it easy to run programs and share resources is called the `Operating System (OS)`

- The primary way the OS makes the system easy to use is through `virtualization`
    - The OS takes a physical resource and transforms it in to a virtual form of itself
    - Basically by providing an api through `system calls`
    - This also allows many programs to be run in parallel (as long as there's hardware support)
    - The OS basically manages the resources so that many programs can access them
    - The goal of resource management is to provide fairness and efficiency


## Virtualizing the Computing

```
int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: cpu <strig>\n");
        exit(1)
    }
    char *str = argv[1];
    while(1) {
        Spin(1);
        printf("%s\n", str);
    }
    return 0;
}
```

- Let's say we run this on a single CPU
    - Not super interesting, it spins for a second than prints out the string that was passed in and repeats forever.
    - The output will look something like this:
    ```
    prompt> ./cpu "A"
    A
    A
    A
    ctrl-C
    ```

- What happens if we run many instances of the program?

    ```
    prompt> ./cpu "A" &; ./cpu "B" &; ./cpu "C"; ./cpu "D" &
    [1] 7353
    [2] 7354
    [3] 7355
    [4] 7356
    A
    B
    D
    C
    A
    B
    D
    C
    A
    C
    B
    D
    ctrl-C
    ```

    - The heck?
    - Even though we only have one CPU, all four programs seem to be running at the same time

- The OS provides the illusion of many CPUs through virtualization
    - It can pretend that there are an infinite number of CPUs

- The ability to run programs in parallel can cause several issues:
    1. Job scheduling (which program should run when?)
    2. How is shared data between processes affected?


## Virtualizing Memory

```

int main(int argc, char *argv[]) {
    int *p = malloc(sizeof(int));
    assert(p != NULL);
    printf(" (%d)address point to by p: %p\n", getpit(), p);
    *p = 0;
    while (1) {
        Spin(1);
        *p = *p + 1;
        printf(" (%d) p: %d\n", getpid(), *p");
    }
    return 0;
}
```
- The `Model of Physical Memory`
    - Memory is an array of bytes
    - To r`read` memory, one must provide and address to be able to access the data stores there
    - To `write` to memory one must also specify the data to be written to the given address

- Memory is accessed all the time when a program is running
    - A program keeps all of its data structures in memory and accesses them through various instructions like `load` and `store`
    - Since each instruction for the program is also stored in memory, so on each instruction fetch we have a memory read

- Again the program above is not very insteresting
    - First it allocates some memory, then prints out the address of that memory
    - It puts zero in to the first slot of the allocated memory
    - For each loop, it delays for a second, then increments the value stored at p and prints the PID
    - The output looks like this:
    ```
    prompt> ./mem
    (2134) memory address of p: 0x200000
    (2134) p: 1
    (2134) p: 2
    (2134) p: 3
    (2134) p: 4
    crtl-C
    ```

- Let's try running multiple instances at once!
    ```
    prompt> ./mem &; ./mem
    [1] 24113
    [2] 24114
    (24113) memory address of p: 0x200000
    (24114) memory address of p: 0x200000
    (24113) p: 1
    (24114) p: 1
    (24114) p: 2
    (24113) p: 2
    (24113) p: 3
    (24114) p: 3
    ctrl-C
    ```

    - Each program allocated memory at the same place but seems to be updating 0x200000 independently
    - This is because each program has it's own view of memory through virtualization
        - So the 0x200000 in program 1 is not the same 0x200000 in program 2 in real, physical memory
    - The OS maps `virtual addresses` to `physical addresses`

## Concurrency

- `Concurrency` refers to several problems centered around working on many things at once
    - The OS is doing so many things at once that it leads to some interesting problems

```
volatile int counter = 0;
int loops;

void *worker(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        counter ++;
    }
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: threads <value>\n");
        exit(1)
    }
    loops = atoi(argv[1]);
    pthread_t p1, p2
    printf("Initial value: %d\n", counter);

    Pthread_create(&p1, NULL, worker, NULL);
    Pthread_create(&p2, NULL, worker, NULL);
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("Final value: %d\n", counter);
    return 0;
}
```

- This program creates two threads using _Pthread_create()_
    - A thread is basically a function running within the same memroy space as other functions with one or more of them active at the same time
    - Each thread in this example is running _worker()_  which just increments the counter over a loop
    - The ouput looks like this:
    ```
    prompt> ./thread 1000
    Initial value: 0
    Final value: 2000
    ```
    - What happens if we run it again?
    ```
    prompt> ./thread 100000
    Initial value: 0
    Final value; 143012 // eh?
    prompt> ./thread 100000
    Initial value: 0
    Final value: 137298 // what in tarnation?
    ```

- How can a program output different values when given the same input value?

- This boils down to how instructions are executed, specifically what order

- There are three instructions to increment the counter: one to put it in to a register, one to increment it and one to save it
    - These are not executed `atomically`
    - So the CPU scheduler can decide to switch which thread it's running in the middle of the increment instruction set

- This is one of the main problems in concurrency

## Persistence

```
int main(int argc, char* argv[]) {
    int fd = open("/tmp/file", O_WRONLY | O_CREATE | O_TRUNC, S_IRWXU);
    assert(fd > -1);
    int rc = write(fd, "hello world\n", 13);
    assert(rc == 13);
    close(fd);
    return 0;
}
```

- There are two kinds of memory: `volatile` and `persistent`

- Volatile memory is DRAM or RAM and it's temporary storage
    - Lost when you turn off your computer

- Persistent memory is a hard disk or SSD
    - Not lost when you turn off your computer

- The software that manages the persistent memory is called a `file system` because it is responsible for storing any files the user creates in a reliable and efficient way on the disk

- The OS does not create a private virtualized disk for each program, it assumes that we want to share the information on the disk
    - This means you can have multiple files open in vim for example

- To accomplish the program above, the program makes three calls the to the OS
    - First it calls open to open/create the file, second it calls write to write the data to the file, and finally it calls close to indicate the program won't be writing to the file anymore
