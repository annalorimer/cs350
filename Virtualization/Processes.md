# Processes

- The `process` is one of the most fundamental abstractions that the OS provides
 - Informally, a process is a running program

- Since a program can't run itself, the OS is what makes the program actually
useful

- An important feature of modern OSes is the ability to run multiple programs
at once
 - These programs should not have to be concerned about wether a CPU is
 available

- So our challenge is to provide the illusion of multiple CPUs despite only
having one

- The OS accomplishes this by `virtualizing the CPU`
 - A program is run, then paused while another program makes progress

- Several pieces required to do this:
 1. Mechanisms (Low level methods or protocols)
 2. Time-Sharing
 3. Policies

## The Process Abstraction

- To understand what constitutes a process, we need to understand `machine
state`. Machine state is:
 1. `Address Space`: The memory of the process (including registers)
 2. `Program Counter`: Indicates which instruction is currently being executed
 3. `Stack Pointer` and `Frame Pointer`
 4. `I/O Information`: ex. What files the program currently has open

## The Process API

- The following functions must be included in the process API

**Create**

- An OS must be able to create a new process

**Destroy**

- An OS must be able to destroy a process forcefully

**Wait**

- An OS may need to temporarily halt a process

### Process Creation

- The OS must first load the process' code and static data in to memory and in to the address space of the process
 - Programs are on the disk in an executable format, so the OS must transfer them to memory

- Modern OSes load programs in `lazily`, only process code and data as it's needed
 - Older OSes will load programs `eagerly` i.e all at once

- The `run-time stack` and the `heap` need to be allocated before the program can be run


## Process States

- A process can be in one of three states at any given moment

**Running**

- The process is running, i.e executing instructions on the processor

**Ready**

- The process is ready to run, but the OS has decided to not run it

**Blocked**

- The process is forced to wait for another operation to complete i.e a call to an I/O device


- A process is moved between the running and the ready states at OS's discretion
 - Moving from ready to running requires the process to be `scheduled`
 - Moving from running to ready means that the process has been `descheduled`
  
