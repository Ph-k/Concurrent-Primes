In this repository you will find two projects, **[Concurrent-Primes](#Concurrent-Primes)** and **[Salad-Maker](#Salad-Maker)**.
Both projects deal with multiple concurrent process that must be coordinated in order to progress towards a common goal. Briefly:
- The goal of Concurrent-Primes is to calculate prime numbers, while communicating via pipes. 
- Whereas the Salad-Maker goal is to make some “virtual” salads, with the processes sharing the "virtual" ingredients on a shared memory segment, protected by semaphores.

*These projects were the programming projects 2 & 3 of the Operating System course at DIT – UoA. Credits to the professor of the course at the time,  [Alex Delis](https://www.alexdelis.eu/).*

# Table of Contents  
[Concurrent-Primes](#)  
- [Summary](#)
- [Compilation](#) 
- [Usage](#) 
- [Brief implementation overview](#) 
-- [Process hierarchy](#) 
- [Source code files overview](#) 
- [ Output format](#)
 - [Useful notes](#) 

[Salad-Maker](#)  
- [](#) 
- [](#) 
- [](#) 
- [](#) 
- [](#) 


# Concurrent-Primes

A hierarchy of processes that communicate in order to find prime numbers in C

## Summary:

This project focuses on creating and coordinating a dynamic number of processes utilizing the appropriate linux/unix system calls *(fork(), exec(), wait(), kill(), mkfifo(), pipe() etc)*. By using these system calls, it creates a hierarchy of processes which is used to calculate prime numbers using different prime-finding algorithms implemented in different executables. The processes hierarchy can also be used as a basis for any kind of concurrent calculation that can be split between different programs.

## Compilation:

By typing `make` on a tty in the source code directory, all the necessary programs will be compiled, creating all the necessary executables. The only requirement is a linux/unix machine running a stable version of the gcc compiler. *The program worked fine on Debian based distros.*

## Usage:

The program can be executed from a cli as `./myprime -l lb -u ub -w NumofChildren`  where:

- myprime: is the *root* executable of the project
- lb: is the first number that will be checked as a prime.  The start of the prime-checking range.
- ub: is the last number that will be checked as a prime. The end of the prime-checking range

- NumofChildren: is the number of child processes the myprime process and each primeManager process will create.

## Brief implementation overview:

### Process hierarchy:

**There are three groups of processes:**

1 **Creator/Root:** The basic executable `myprime` is the root of the process’s hierarchy and father of all internal node processes, which in turn create the leaf node processes.

Apart from creating all the needed internal node processes, it also splits the prime-finding range according to the number of internal nodes so that each internal node is responsible for finding a unique sub range of primes. Finally, it collects the found primes *(and the time taken to find it)* from all his children using pipes.

2 **Managers/Internal nodes:** The internal-node processes `primeManager` are responsible for creating  worker/leaf-node processes which together perform the calculations needed to find the primes in the primeManager's sub-range. They also collect the found primes and needed execution time from the leaf node processes using pipes, and compose them into a sorted list. When the child processes are done calculating, the sorted list of results is send to the root node.

3 **Workers/Leaf nodes:** As implied above, the leaf-node processes implement the algorithms which find the primes in a given sub range *(which is a sub range of the internal’s node sub range)* and send them via pipes to the managers/Internal nodes along with the time the calculation took. These processes can be any of `prime1` `prime2` `prime3` executables, which are selected using round dropping in the internal nodes.

----

The number of children processes the creator and each manager spawns is determined by the `NumofChildren` flag on `myprime`, the depth of the hierarchy is always two.
For example, here is a schematic representation of the process hierarchy for 3 child processes:

![Untitled Diagram (1)](https://user-images.githubusercontent.com/17359348/182947379-9757234d-0843-44a6-888b-f87fe6f3d068.png)


Furthermore, each leaf-node process sends a USR1 signal to the root process *(who of course has the according signal handler)* to inform him that he has finished working 

## Source code files overview:

`myPrime.c`: Implements the already mentioned myPrime/Creator processes. The creation of the child processes is done using the fork() and execlp() system calls. The data from the children processes (prime numbers, execution time for leaf nodes etc) is collected through unique pipes for each child process.

`primeManager.c`: Implements the internal-node processes. The program takes as command line arguments the: *1.* range of numbers which will be split and searched for primes from the leaf-node processes, *2.* number of child processes, *3.* the file descriptor of the pipe used to communicate with the root process, *4.* his child number, used as a serial number to determine which child of the root process he is, *5.* the root/myPrime process id to be passed to the leaf nodes. Also, as mentioned, it creates the leaf-nodes, and composes the results of the child process in a sorted list which, when done, is forwarded to the root process.

`prime1.c` & `prime2.c` & `prime3.c`: Each of these programs implement a prime funding algorithm from faster to slower, from trivial to more sophisticated. The two first algorithms were given from the professor alex delis, and the third is a prime finding implementation of my own.

Regarding everything else, the programs work the same way. The all take the same arguments (prime-finding range, pipe file descriptor, process id of root/myPrime process to send the USR1 signal), and as they find the prime numbers they send them to their respective parent using their unique pipe.

`list.c`/`list.c`: Contains the implementation of a C linked list, customized to fit the needs of maintaining and inserting in sorted order prime numbers and the time needed to find them.

`utilities.c` & `utilities.h`: Some useful functions that help maintaining a clean code base by providing a level of abstraction, and preventing code duplication. More in the source code comments.

### Output format:

All the output is printed from the root/myPrime in the following form:

    OUTPUT (per invocation of program):
	Primes in [lb,ub] are:
	result1 time1 result2 time2 result3 time3 result4 time4 ... resulti  timei...
	Found *n* prime numbers in total
	
	Min Time for Workers : mintime msecs
	Max Time for Workers : maxtime msecs
	Num of USR1 Received : numUSR1rec-by-root/number-of-workers-activated
	Time for W0: W0-time msecs
	Time for W1: W1-time msecs
	Time for W2: W2-time msecs
	Time for W3: W3-time msecs
*---Credits to professor alex delis for the above output format ([link](https://www.alexdelis.eu/k22/formatted-output.f20-prj2-v1.txt))---*

 


### Useful notes:

- All of the `prime1,2,3` and `primeManager` executables need to accept arguments, but apart from a check that the correct number of arguments were given, no further checks or error handlers are programed since we assume that the arguments are always well given from the parent process *(and not from the user)*.
- The inter-process communication is done in a non-blocking manner. 
- At `utilities.h` you will find the communication norm *`struct pipeMessage`* used to transfer different types of data through the pipes.
