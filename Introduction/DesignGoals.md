# Design Goals

- There are several goals we want to achieve when we design an OS

## Abstractions

- This makes the system more convenient for the user and are fundamental to all of computer science

- Abstactions make it prossible to write a large program by dividing it in to smaller chunks
    - i.e write a program in C without having to think about assembly code or how to build the processor

## Performance

- We want to minimize overhead

- Virtualization and making the system easy to use is essential, but it has to be balanced with performance

## Protection

- We will want to provide protection between applications and as as between the OS and applications

- Because many programs can run at the same time we want to prevent accidental or malicious bad behaviour

- We definitely don't want an application to do harm to the OS itself

## Reliability

- The OS must not stop running otherwise all of the applications will fail
