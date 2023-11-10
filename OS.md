Control registers:
1. ia - Instruction address register contains the address of the next instruction
2. psw - Program status word -
    a. 00 - System Mode / Masked interrupts
    b. 01 - System Mode / Enabled interrupts
    c. 10 - User Mode / Masked interrupts
    d. 11 - User Mode / Enabled interrupts
3. base - added to all addresses when in user mode (part of physical memory the instructions and data can be accessed)
4. bound - upper address limit (else program error interrupt with ip=2), done before base added for user mode
5. iia - During an interrupt, Interrupt Instruction Address Register stores the value of ia register before an interrupt while ia will have address of interrupt handler. 
6. ipsw - During an interrupt, Interrupt Program Status Word stores the value of psw before an interrupt while psw loaded with 0
7. ip - Interrupt parameter register contains data about last interrupt
8. iva - Interrupt vector address register stores location of Interrupt Vector Table
9. timer - Interval timer register decrements once every microsec till reaches 0, when a timer interrupt is generated and storing 0 in this register clears any pending masked timer interrupts

System Mode - 
All addresses are physical
All instructions can be run
Interrupts can occur if psw[1] = 1

User Mode -
Logical addresses
Instructions forbidden to modify control registers (else cause program error interrupt ip=1)

Interrupts - immediate transfer of control caused by an event in the system

Example of Interrupts-
1. Program Error -
    a. ip=0 : undefined instruction
    b. ip=1 : illegal instruction in user mode
    c. ip=2 - address breached the bound

Interrupt handling by hardware -
1. psw -> ipsw
2. 0 -> psw
3. interrupt parameters -> ip
4. ia -> iia
5. 4 * (interrupt number) + iva = interrupt vector entry -> ia

Return for Interrupt (rti) -
1. iia -> ia
2. ipsw -> psw
3. normal execution

Masked Interrupts - recorded but not processes

System Call instruction generates an interrupt that causes the OS to gain control of the processor, instruction set of the OS virtual processor

System Calls-
1. Program executes system call instruction
2. ia -> iia & psw -> ipsw
3. 0 -> psw (system mode, interrupts disabled)
4. 1st instr of syscall interrupt handler (in syscall interrupt vector loc) -> ia
5. Syscall handler completes and executes rti (iia -> ia & ipsw -> psw)
6. Normal execution

File and I/O Syscalls - 
1. Open - Get ready to read/write a file (returns file pointer)
2. Create - create a new file and open it
3. Read - Read bytes from an open file
4. Write - write bytes to an open file
5. Lseek - change the location in the file of the next read/write
6. close - done reading/writing a file
7. unlink - remove file name from disk
8. stat - properties

Process Management System Calls -
1. CreateProcess - create a new process
2. Exit - terminate the process making the system call
3. Wait - wait for another process to Exit
4. Fork - create a child process
5. Execv - run a new process

Interprocess Communication System Calls -
1. CreateMessageQueue - create a queue to hold messages
2. SendMessage - send a message to the message queue
3. ReceiveMessage - receive a message to the message queue
4. DestroyMessageQueue - destroy a message queue

File & I/O Syscalls - 
1. open(filepath,flages) -> fid : creates open file connected to a file
    a. flag=0 -> reading
    b. flag=1 -> writing
    c. flag=2 -> read/write
2. creat(filepath,mode)-> fid : creates file and connects to open file
3. read(fid,buffer,count)->count : Reads bytes from open file
4. write(fid,buffer,count)->count : Write bytes to open file
5. lseek(fid,offset,mode) -> offset : Move position of next read/write
    a. moveMode=0 -> 0 as base (beginning)
    b. moveMode=1 -> current file location
    c. moveMode=2 -> current file size (ending)
6. close(fid)-> code : Disconnect open file from file
7. unlink(filepath)-> code : Delete the file

Files are just containers of data
Open Files are dynamic objects which allows byte read/writes into the file (an interface of the file)

Programs -
Static object that can exist in a file, and contains sequence f instructions

Process -
dynamic object with program instructions in execution, existing for a limited span of time
Each process has a save area where it saves its context (register values)

Differences between Pipes and Message Queues:
Pipes -
1. no fixed sized messages
2. created till the lifetime of the two processes, once closed cannot be opened
3. Connection between two processes (FIFO)
4. Can read data all at once

Message Queues:
1. Fixed size queues
2. stored in the RAM, can be closed and opened multiple times(more memory persistent)
3. No connection between two or more processes
4. Message Queues retrieves data individually

Message Queues use special identifiers rather than file descriptors, so don't rely on file I/O
Message queues do not garuantee FIFO, processes can specify the order
If a process requests data less than the Message Queue data size unit, it will receive nothing
For two processes to communicate with a Message queue, they need to have a common ancestor

A POSIX Message Queue is a prioirity queue
It is removed when all processes using it are closed
when receiving messages, msg_len must match the size of a message.

can be used only when used for writing

Named Pipes: 
Pipes are named by the file system
THey have some kind of existence even with no process using them
if the two process know the name of a pipe, they can open it and use it as an open file

Named Message Queues:
No name, and no existence till they are created

SOS

Simple OS has 2 major subsystems:
1. Process Management Subsystem
2. Disk Management Subsystem

Message Queues in SOS are implemented by two Queues:
1. message_queue to store messages
2. wait_queue to store processes waiting for messages

Process Management Subsystem-
handles process abstraction
maintains process tables, creates message queues, system calls, dispatches processes to run on processor

Disk Management Subsystem-
handles disk abstraction
communicates with the disk hardware and handles disk interrupts
includes a disk driver which accopts disk operations and schedules them on the disk

Save Area -
Records hardware context of a process (contains all the registers)

Process Descriptor
Data structure which records the state of a process (register and process state) (slot allocated, time left till switch, state (Running, Ready, Blocked, Save Area

Interrupts-
1. System Calls
2. Timer
3. Disk
4. Program Error

System Stack -
It doesn't hold any procedure flow of the OS
It is just used as a data structure to interact between user and kernel mode

Timer Interrupt handler:
Timer is hardware interrupt
It backups context into the process's save area (Half context switch)
It calls the dispatcher

Dispatcher - 
1) Selects Next process - It loops through the processes (starting from the current process) to find a ready process 
2) Runs Process - It then sets its state as RUNNING, sets its timeQuantum, loads its save area context

Context Switching -
1. Hardware : ia --> iia, psw --> ipsw
2. Interrupt Handler : iia. ipsw --> Save Area
3. Interrupt Handler : Rest of registers --> Save Area
4. Interrupt Handled by OS
5. Dispatcher called (loads the next process state)
6. Save Area --> iia, ipsw
7. Rti

System Initialization
OS Initialization (Setup System Stack, Interrupt Vector Area, Process Descriptor Table, Message Buffers, Message Queues)
Call Dispatcher
Process 0 - idle
Process 1 - init

System Call Interrupt Handler
1. Save Context of caller
2. Switch case based on required system call
3. Move to the respective system call handler after getting the arguments
4. Dispatcher()

Disk Handling
Disk Queue maintains a list of required Disk Reads/Writes left to run
DiskRead/DiskWrite will call DiskIO to put a request on the Disk Queue and set the state of the process into waiting
When the hardware does the required action, it will call the Disk Interrupt Handler (hardware prompted) which will settle things.

DiskRead - 
DiskWrite -
DiskIO(int command, int block, char *buffer) - Creates a new DiskRequest(command, block, buffer, pid), inserts it into the Disk Queue, sets process state waiting and calls the ScheduleDisk (for disk action)
ScheduleDisk(void) - if disk busy, return else execute the command
DiskHandler(void) - Save current context, set waiting process as ready, call ScheduleDisk and Dispatcher
DiskBusy(void) - Check disk status register and return if it is busy (Hardware Interface)
IssueDiskRead(int block, char *buffer, int enable_disk_interrupt) - (Hardware Interface)

OS System Calls / OS processes have very small state size (no memory, only few specifying details)

Message Waiting -

Receiver End -
1. The Receiving Process will start a ReadMessage Syscall specifying its Message Queue
2. The OS will check the Message Queue:
	i. If message present, then store it into the read buffer and rti
	ii. Else, Process and the system call is blocked (the receiver pid and buffer address stored in the wait queue)
3. Once Awoken, the process is completed

Sender End -
1. The Sender Process will start a SendMessage Syscall specifying its Message Queue
2. The message is added into the Message Queue
3. If there are any Receiving process, then they are awakened and given value

Suspending System Calls -
1. Store the system call state for later use
2. Make sure is will be awoken by some other process

Disk Scheduler -
1. Save PID in Disk Queue
2. Disk Interrupt will awaken the process

OS as an Event Manager -
System that responds to events (Reactive Systems)
It is passive entity, responding to various events (either generated by processes or external devices)
Events generally change the process states of processes
Can be described by state machine
State of an OS is the state of its data structures

Process Table -
Its entry is Process Descriptor for each process
Process Descriptor contains - PID, Process Name, Memory allotted, Open Files, Process state, User name, Save Area, Parent Process

SOS Process Descriptor:
1. slotAllocated --> whether memory alloted or not
2. timeLeft --> time left before interrupt
3. state --> RUNNING/WAIT/READY/TERMINATED
4. inSystem --> Whether it has some kernel threads/system stack/no of save areas in system stack
5. lastsa --> last save area
6. sstack --> system stack

Ready List -
Set of all processes set to ready state

CHAPTER 6

Parallelism

Two Processors-
Private - Set of registers, Timer (call and receives its own interrupts)
Shared - Memory, Disk

* Issues with OS per processor-
(Each OS will have its own process table, memory segment)
1. Increased memory usage to store 2 OSes
2. Difficult Inter OS Process communication
3. Which controls the processor and disk of them (Master OS) ? *

SHARED OS - 
Only 1 OS Code
System Initialization done by only one processor (which initialises its data then starts the second processor)
Two processors run mutual exclusively at the same time
Each processor has its own
a. Set of registers
b. System Stack (better coordination)
c. current processes
d. Timer

Global Data -
a. Process Table (Not good for two individual process table as one may be full ready other not)
b. Disk Queue
c. Message Queues

Race Conditions -
Where two processes are interacting in some way, and the relative speed at which they run will affect the output
The last process's effect will prevail and overrun the others
Can only occur in cases where there is some communication between processes and are running parallelly
To solve it --> make the execution of critical section as serial operations

Atomic Action -
Non overlapping actions
Its intermediate state cannot be seen by any other processes
Ex - memory R/W

Critical Section - Section of program with shared memory

Exchangeword - 
To make reading old process state and writng the new value into an atomic action
Exchanges the values from a memory cell and hardware register (read from memory cell and write into it)
If two processors run ExchangeWord on same memory cell, then the cell will hold either of the values (the only way to do this both processes run non overlapping manner)
To gain access of the memory cell, a process calls the memory module over the system bus.
So the bus H/W unit fixes a processes as the bus master for some amount of cycles (1 cycle for read/write) and lets it run its atomic actions uninterrupted
This is used in the case of sharing Process Table (with the non-using one in a Busy Wait)

Process Table - 
Being a shared resource, we need to protect it from illegal accesses
To use it, a processor must check a ProcessTableFreeFlag whether the process table is in use or not
Given a processor is using the process table, it is given complete access and closes as per its need
The other processor will be in a Busy wait, continously checking the value of ProcessTableFreeFlag

Problem with Busy Wait is waste of time

Spin Lock - 
Variable protecting a shared resource and its state spinning (ExchangeWord) between itself and the processors
The ProcessTableFreeFlag above is a Spin Lock

Dispatcher -
We fix the process table to ourselves during its run

Message Queues -
Spin Lock - Key per Queue (Mutual Exclusion)
Once a processor receives a key, it can enter the room and access the Message Queues
The other process will spin the Key
After process 1 exits, the other enters

THREADS

Software Abstraction of Multiprocessor Computer
Lightweight Processes
Share memory and can run parallelly
More efficient,lighter and faster than processes (as no issues or delays with memory)
They don't use system calls to communicate, instead using memory
Provide parallellism in single process (Process parallellism is between two different processes)
Increases responsiveness 
Will have its own system stack and set of registers
Possible to have multiple threads to share the same code space

Each thread is executing a subset of the same instructions but in different execution (different contexts)
When a process is created, an initial thread is also created.
The process will exit only when last of its threads has exited (threads can exit any time)

Thread System Calls - 
1. CreateThread (startAddress, stackBegin) : thread_id -
	startAddress will give starting Address of execution
	stackBegin will have its stack beginning at this memory location
2. ExitThread (returnCode) -
	Exits the thread
	If last thread, calls ExitProcess and returns back to Wait syscall of Parent Process
	Else, returns back to parent Thread calling WaitThread
3. WaitThread (int thread_id) -
	Parent thread will wait for new thread

Process Table:
Each entry will have memory bounds and the head to the linked list of threads (no more process state)

Thread Table:
1. register save area
2. parent pid
3. thread state
4. pointer

The Dispatcher will run threads instead  of processes
No more ExitProcess (to exit a process we have to exit all of its threads)

User Threads
User process can have its own threads (and a scheduler to manage them)
More efficient than lightweight processes
Not recognised by the OS
If one user thread is blocked, the whole process blocks

Kernel Mode Processes -

OS Threads
In the simple implementation, we had made the OS wait for Message Queues, or Disk Queues.
This is done by saving the process context and blocking it
Better than this, we allow threads to run inside the OS Address Space, which will do all of the processing and interrupt handling.
Each Process is given its own Per-Process System stack which will store the context (save area) for each interrupt, and its kernel threads refer to the top of the stack (latest context)
Each User process will also have its own set of kernel processes
Each new save area will be linked to the previous save area.
This speeds up the procedure.
Each save area will have its first word as a pointer to the previous save area
Each process will a counter as inSystem which will count two things:
1. No of save areas stored in the system stack
2. No of threads?

With Kernel Processes, we can now read from the disk during CreateSysProcess

All Interrupt Handlers will do the following -
1. save the register state on the per-process system stack
2. Keep the save areas in the system stack linked
3. Update inSystem

SystemCall Interrupt Handler:
1. Save Area Updation -
	i.	We will first acquire the memory chunk from the stack for the new save area
		(We are substracting the size of Savearea + 4 because it is a stack, so bottoms up)	
	   	If the process is entering into kernel mode for the first time, it will allocate the last (empty) slot in the linked list (stack)
	ii.	We store the registers, timeLeft and the pointer to the last save area in this space (as this save area forms the new top)
	iii.	We then push the new save area into the System stack (making it as the top)
	iv.	We shall link this new save area to the last save area
	v.	We increment the inSystem
2. Interrupt Running -
	i.	Starts by loading 2 into psw register
	ii.	Switch based on the system calls

Changes to Send/Receive Message Queues:
1. SendMessageCall(char msg, int q) - No longer has to send the message directly to the receiver.
	i.   Get a Message Buffer and copy the user msg into it
	ii.  If there is some receiver waiting in the wait queue, then wake it
	iii. Insert the message into the message queue
	iv.  Call Dispatcher
2. ReceiveMessageCall(user_msg, q) - Check if the message queue is empty, block itself else take the message
	i.  If Message Queue is Empty, state is blocked, insert itself into the wait queue and Switch
	ii. Else, transfer the message from the queue to the user msg buffer

Switching Kernel Processes
SwitchProcess(pid) - called when some Kernel process wants to wait (Switch processes)
1. Its context saved in the save area
2. State made Blocked
3. Dispatcher()

Difference between SwitchProcess and Dispatcher :
1. SwitchProcess will switch between kernel processes only while Dispatcher will switch between both user and kernel processes
2. SwitchProcess will add a new Save area into the system stack while Dispatcher will remove a save area

Each time a SysCall or SwitchProcess is made, a save area is added to the stack
Whenever we are modifying the save area, we cannot allow interrupts to occur.

Example : Process wants console input

1. Calls Systemcall("Console Read")
2. Kernel mode switch
3. Save Area made and pushed into the System Stack (SAVE_AREA 1)
4. Console Read case is chosen
5. Console Read picks up the pid and user_msg buffer to read into
6. It schedules the process in the terminal wait queue
7. It calls the SwitchProcess
8. SwitchProcess will create a new Save Area and push into the System Stack (SAVE_AREA 2)
9. SwitchProcess will call Dispatcher
10.After the terminal input is entered, Console Interrupt is raised
11.This process is put into the ready state
12.When Dispatcher picks it up for scheduling, it will reload SAVE_AREA 2 and run the process
13.This Save Area will have the ia set in Console Read, so it will run and finish it
14.It will exit Console Read case, exit Systemcall and call Dispatcher
15.The Dispatcher will schedule it again (as it in state Ready), reload SAVE_AREA 1 and run it further

Dispatcher Kernel Process

RunProcess(pid) -
1. Remove the latest save area from the system stack (and deccrement the inSystem)
2. Make the last save area as the top
3. Load the save area contents into the registers
4. Rti

Kernel Mode Only Processes
Processes that run only in OS
Example: ScheduleDisk in the disk Controller which looks for any disk requests in the disk queue and schedules them to the disk
These processes can block themselves and are invoked by other processes

Advantages of Kernel Processes -
1. Easy waiting in OS
2. All states are preserved and processes can wait anywhere in the OS
3. Allow interrupts in OS

Disadvantages of Kernel Processes -
1. Memory : Every Process needed its own User Stack and Kernel Stack

MUTUAL EXCLUSION

Solution 1 - Disabling Interrupts :
Useful only in uniprocessors
Very Fast
Doesn't use busy waiting

Solution 2 - ExchangeWord :
Hardware help needed
Requires Busy Waiting
Works with multiprocessor
Best solution for processors sharing memory

Solution 3 - Software :
Dekker Solution was first (complicated and wrong)
Peterson Solution is simple and used
Requires Busy Waiting
No Hardware Assistance
Best for distributed systems with no central control

Peterson Solution :
If Process A & B want to do some processing on a Shared Critical Section, their control flow is:
1. EnterCriticalSection(pid)
2. PerformCriticalOperation()
3. LeaveCriticalSection(pid)

Assumptions about Processes A & B:
1. Both cannot be in critical section together
2. They cannot wait forever inside EnterCriticalSection
3. They spend only finite time inside PerformCriticalOperation() and do not fail to Leave
4. They can spend any time outside Critical Section

Algorithm:

Global Data -
process PIDs = 0 || 1;
interested[2] = {FALSE, FALSE};
turn = -1;

EnterCriticalSection(this_pid) {
	other_pid = 1 - this_pid;
	interested[this_pid] = TRUE;
	turn = this_pid;
	while(interested[other_pid] && turn == this_pid)	pass;
}

LeaveCriticalSection(this_pid) {
	interested[this_pid] = FALSE;
}

Assumption of Message-primitives : Mutual Exclusion in OS

CHAPTER 7 INTERPROCESS COMMUNICATION PATTERNS

3 Approaches in communicating:
1. Message Queues
2. Pipes
3. Shared Memory

Process Competition - When processes compete with each other for a common resource
Process Cooperation - When processes work together to achieve a common and complicated task

Logical Resource - entity defined by the OS
Serially Usable Resource - can be used by only one process at a time and can be reused by other processes

Attaching and Detaching Message Queues:
To name a Message Queue as per file naming system, we can attach a Message Queue instead of creating and then detach it
First attaching will create it as an OS data structure, and subsequent attaches will return an internal identifier to it

AttachMessageQueue(char *msq_name) -
1. If there is no message queue named msq_name, it is created as "Message Queue File"
2. Attach count for it is incremented
3. An identifier to the message queue is returned which is used to send/receive messages (msq_id)

Message Queue Identifiers are OS-wide identifiers and have the same meaning across all processes.

DetachMessageQueue(int msq_id) -
1. Attach count is decremented
2. If Attach Count == 0, destroy message queue

When a process is exited, all attached queues are detached.

N User Process Mutual Exclusion:
We will use a message queue for both processes to communicate

1. We set up a Message Queue (by AttachMessageQueue)
2. We initialise it with some dummy message (or seed it) via one process.
3. When a process wants to use the shared resource, they will receive the message from the queue
4. This message acts like a ticket to use the resource
5. Once it is done using the resource, it will send the message back into the queue
6. When one process is using the resource, the queue is empty, so other cannot use it

This implementation depends on the dependability of processes (that they will first wait to receive a message and then try to access the resource)
To prevent the non-dependable processes, we use monitors.

Assumption of patterns - the message-passing primitives present in OS.

Signalling -
When one process acts as an informer to the user by giving signals depending on the events
It is not symmetric, as the sender doesn't wait for the receiver
It involves only one interaction

Interprocess Signalling pattern - one process/daemon told to signal an informer process
In this case, both will share the same message queue, and the daemon will send a message once a task is done
Then the informer will pick it up and process it.

We don't use busy waiting in case of OS as too much time is wasted 

Rendezvous -
Two process simultaneous communication
Two one-way signals
Symmetric, as both sender and receiver wait for messages

If we have processes A & B:
There are 2 Message Queues between them
Each process uses one queue to send and one queue to receive
As both queues are independent, it can be done simultaneously
The receiver doesn't look at the message contents

A ======> B
A <=====> B

For Many Process Rendezvous:
We will have 1 central process called Coordinator (Server)
All other client processes will communicate with the Coordinator, which will communicate with them individually
There will a common message queue called coordinator_queue through which client queues can send their message
Each process will have their own message queue (defined by queue identifier) through which the coordinator sends back the message

Initially, each client process will send their queue identifier to the coordinator via coordinator_queue, who will note it down in an array.
Then the coordinator will send a message through the individual queue.

Producer - Consumer :
Here the first process will produce an output which the consumer will take as an input
Both processes will have their own functionality
They communicate via a pipeline
Almost all process cooperation can be seen as a variation of Producer - Consumer pattern (exception when the task is split into disjoint parts which can be done parallelly)

Example - 
cat OS.txt | grep process

To Implement a Pipeline
It is a one-way signal in a loop (multiple interactions)
Both producer and consumer share a message queue
Producer can send a message, while consumer can receive

The Producer can get ahead of the consumer via the OS buffering the messages (queueing them)
This is useful when either of them are bursty (work at an uneven rate)

Limited Buffering:
In the case where the size of queues is limited, we can use another buffer queue to store messages
So the producer will send a full buffer of messages to the consumer who will signal the producer that they have received the message

There are two queues - producer_queue & buffer_queue
1. Consumer will initialise the interaction by sending BufferLimit no of messages via the producer_queue (for each free buffer)
2. If the producer_queue is not full, the producer will send the message into the buffer_queue
3. Once it is full, the full buffer_queue content is dumped upon the consumer
4. The consumer will receive and reply that they have received and a buffer is now free.

There are 2 producer-consumer relationships:
The consumer consumes full buffers and produces empty buffers
The producer produces full buffers and consumes empty buffers

Rendezvous - When the BufferLimit is 1

Multiple Producers and Consumers
Same procedure

Client Server Model -
Server Process owns resources which are used by the client processes
Server manages requests and accesses to the resources
Client processes need to know the common queue to send the service requests
The server doesn't needs to know the individual queues to each client and just waits for requests
A service request that requires response will include name a queue

File Server:
Provides range of file services
It further uses the services of Disk Block server to get the actual file processing done
It also maintains a Pending Requests Table for pending requests while the Disk Block server responds

Variant of Producer-Consumer model

Differences between Server-Client Model and Producer-Consumer Model:
1. Client doesnt send data to server to be processed, it sends to retrieve
2. More assymmetric
3. Server doesn't know about clients
4. Connection is temporary
 
Database Access and Update Model -
Common database with several readers & writers
Readers can share access to the database with each other, but not with writers (Reader-Writer Problem)

If we have readers reading the database, then new readers can enter immediately
However, writers must wait.
If one writer is writing, then all must wait.

The database has its own main queue named coordinator_queue
Each of the client process have their own individual queue named pid_queue asto get response from the server
The pid_queue is called Private Scheduling Queues
The Database server will schedule the access of its data.

Reader Process -
1. Send ReadRequest to Database via coordinator_queue alongwith pid1_queue identifier
4. Wait for response from the Database via the pid1_queue
5. Start Reading
6. Send EndRead Request

Writer Process -
1. Send WriteRequest to Database via coordinator_queue alongwith pid_queue identifier
4. Wait for response from the Database via the pid_queue
5. Start Writing
6. Send EndWrite Request

Database Server -
Loop(True):
1. ReceiveMessage from the coordinator_queue.
2. Decide the type of request

Requests :
1. ReadRequest -
   i.  If there are no active or waiting writers then allow reader to read
       a. Keep a count of no of active readers
       b. Send message to the reader that it may proceed (via pid_queue)
   ii. Put the reader into ReaderQueue

2. EndRead -
   i.  Decrement no of readers
   ii. If No readers left and there are waiting writers
   	a. Increment no of writers
	b. Extract latest writer from the WriterQueue
	c. Send message to Writer to proceed (via pid_queue)

3. WriteRequest -
   i.  If there are no active writers/readers then allow writer to write
       a. Keep a count of no of active writers
       b. Send message to the writer that it may proceed (via pid_queue)
   ii. Put the writer into WriterQueue

4. EndWrite
   i.  Decrement no of writers
   ii. If there are waiting readers
   	a. let all readers to read ( by extracting from the queue and sending approval via pid_queue)
   iii.If there are waiting writers
   	a. Extract latest writer from the WriterQueue
	c. Send message to Writer to proceed (via pid_queue)

Why we did not use a Coordinator process in Database Model -
1. Multiple Readers can read concurrently.

Problem -
A Continous stream of readers can complete prevent a writer from ever gaining access
Solution : When a reader arrives when there is already a writer waiting, it is made to wait and when the last reader leaves, the writer is given access

CHAPTER 8 : SYNCHRONISATION & SEMAPHORES

Synchronisation is when one process waits for a notification of an event that will occur in another process

In Signalling Pattern, we use synchronisation -
When one process reaches a signal point, it waits for the other to reach the signal point (implemented by the message queue)
In Rendezvous Pattern, both processes synchronise together
In Producer-Consumer Pattern, they are kept synchronised so that producer is not too ahead of consumer
In Client-Server Pattern, A single consumer (server) synchronises with the requests of the producers (clients)

Signalling is the most basic type of Program Cooperation
Mutual Exclusion is the most basic type of Program Competition

There will be 2 levels of mutual exclusions:
1. OS level - implementation of message-passing through mutual exclusion
2. User level - implementation of mutual exclusion through message-passing

Semaphores

In OS, Semaphores are used for synchronisation
They are also named but not present in file system and its identifiers are global
A semaphore has two Operations : Wait & Signal
All processes that attach to the same global identifier semaphore will operate on the same semaphore
Each process has its own local identifier for a semaphore

A Binary Semaphore has only 2 states : Locked/Non-locked
It is used to enforce mutual exclusion (so also called Mutex)

A counting Semaphore has n states, specifying no of processes using it currently

AttachSemaphore(int sem_id) -
1. Semaphore with sem_id looked up, and a local identifier is returned
2. If not found, semaphore is created
3. Semaphores are created in busy state

Wait(sem_id) -
1. We get the semaphore from its sem_id
2. We set up a spin lock on the semaphore
3. If another process calls the semaphore while it is busy, it is blocked and added to the semaphore queue
4. Else, the process is free to use the semaphore (and it is made busy)

We use a spin lock on the semaphore to ensure no other process sharing it can use it at the same time

Semaphore not busy? (busy state and return) else (block syscall and put callee into semaphore waiting queue)
If a process calls Wait on a busy semaphore (which means it tries to lock an already busy semaphore) then it is set to block state and put into a wait queue.

Signal(sem_id) -
1. We get the semaphore from its sem_id
2. We set up a spin lock on the semaphore
3. If the semaphore queue is empty, then semaphore is set not busy.
4. Else we extract a process from the queue and unblock it to use the semaphore

Semaphore Queue empty -> no process were waiting to use the semaphore. So the semaphore is truly free.
Signal will free a process from the semaphore and let it open for other waiting processes, so if there is a blocked process in the wait queue, it will unblock it.

DetachSemaphore(sem_id) -
Callee no longer using this semaphore

Wait and Signal are atomic actions
Wait lets the process use the semaphore and Signal frees the semaphore from the process
Thus checking setting state of a semaphore is atomic.

Comparison with Semaphores and Message Queues:
1. Wait(Semaphore id)   = ReceiveMessage(Msq id)
2. Signal(Semaphore id) = SendMessage(Msq id)
3. No of Message Queues = No of Semaphores

Two Process Mutual Exclusion using Semaphores:
1. The processes share a semaphore (which is used via AttachSemaphore).
2. One of them initialise the semaphore as being not busy
3. When a process wants to enter a critical section gaurded by the semaphore, it will call Wait(sem_id) and lock the semaphore (or stand in the semaphore queue)
4. If it got access, it will proceed with its work.
5. It wil then send a Signal call to unlock the semaphore.
6. Finally, the semaphore is detached.

Rendezvous Mutual Exclusion using Semaphores:
Both share a pair of semaphores.
A --> sem_a	B --> sem_b
1. Each will Signal their respective semaphore to say that they are ready to proceed. 
2. Each will Wait the other's semaphore to wait until the other is ready to proceed.

Producer-Consumer Mutual Exclusion using Semaphores:
Two semaphores: Producer_semaphore & Buffer_semaphore

Producer Process:
1. Wait Producer_semaphore
2. Signal Buffer_semaphore

Consumer Process:
1. Signal Producer_semaphore (initialise)
2. Wait Buffer_semaphore
3. Signal Producer_semaphore

To use a Counting Semaphore:
It will have an additional attribute as count, which will count no of processes currently using it.

Wait(Semaphore id) -
1. We get the semaphore from its sem_id
2. We set up a spin lock on the semaphore
3. If its count > 0 (there is still space for one more process to use the semaphore), then decrement count
4. Else, the semaphore is free -> get new process from the wait queue and block it
5. Unlock the semaphore

Signal(Semaphore id) -
1. We get the semaphore from its sem_id
2. We set up a spin lock on the semaphore
3. If the waiting queue is empty -> increment count (here count initially would be 0, we make it 1 so a process later able to lock it)
4. Else, extract the process from wait queue and unblock it

Advantages of Semaphores over Message-Passing:
1. No memory allocation required.

Semaphores are more suitable for Shared Memory
Message Queues more suitable for IPC between processes not sharing

Scheduling:

1. First-In-First-Out Scheduling:
Queuing
Treats all processes equally
Starvation is not possible
No concept of urgent process priority
All are assigned a Queue ID to identify, service in order of ID

2. Shortest-Job-First Scheduling:
Finish all the quickest jobs first
Has the lowest average waiting time
Utilitarian (Good for the group over individual)
Multiple queues : 
	a. Express queue for very quick processes
	b. Normal queue for regular processes

3. Priority Scheduling:
Assign Priority to jobs, based on urgency, or need, or their importance wrt other processes
Priority Queue

4. Round-Robin Scheduling:
Preemption - The ability for a process to halt its execution midway and allow other processes to be scheduled.
Once a process is done, the control passes to the next process (via PID) in a cyclic manner
Thus the control passes through all of the processes in a preemptive manner
Relay Scheduling

Preemptive Scheduling Methods:
From the POV of the processor,
1. Tt gets a certain process for a certain time period
2. It executes its instructions
3. The process finishes execution (for the time being) in the following cases:
	a. I/O Wait
	b. Disk Wait
	c. Message Wait
	d. Time Slice over
4. Then the process is added to the process queue as a 'new' process to the scheduler

Deciding the Time Quantum:
Time Quantum is the time given to the process in the processor
This time should be more than the time taken to schedule process work (cost of preemption)

1. Static - fix a value enough
2. Adaptive - Based on a fixed value and size of the process queue, decide the time slice

We can measure the time slice wrt preemption cost: 1 Time Slice ≡ N preemption cost
This provides the lower bound for time slice


Policy vs Mechanism in Scheduling

General Scheduling Mechanism:
Present in the Kernel
We have 3 Queues (Multiple Queue System in general) (q₁, q₂, q₃)
Each queue has 2 parameters: 
	a. Time Quantum - Q
	b. No of times a process has passed through the Queue - N
The last queue has no N.
N and Q are parameters

We choose the jobs to schedule following the procedure:
1. If any jobs in q₁, select one
2. Else, choose from the next one
3. Repeat for all the queues
Each job is given the time quantum appropriate from the queue it has come from.

Job enqueuing:
Based on its values of N's, it will get scheduled in any of the queues.

Scheduling Policy:
It chooses parameters for the mechanism
It can also increase the no of queues
Present in the OS as the dispatcher module

1. N₁ = ∞, Q₁ = ∞ →  FIFO
2. N₁ = ∞, Q₁ is Finite →  Round Robin
3. Q₁ & Q₂ are finite, N₂ = ∞ →  Two Queue System

We can have a Scheduler Process which wakes up frequently to examine the performance of the system.
It can then make the required adjustments (It would have to bypass all processes, stop the running process to do this)
Else we can set a limit on some particular values (temp, time,mem,etc) and use it to trigger a change of the parameters

Deadlock
(Analogous to Ant's Death Cycle)
(Complete Traffic Jam from all sides)
A cyclic waiting order of process where each process is waiting for a resource which another process in the cycle is holding, which in turn is waiting
These processes will wait forever
Sometimes, messages may get lost in transit, which will lead to a deadlock.

Conditions for Deadlock to occur:
1. Resources are not preemptable
2. Resources are not shared.
3. A process can hold one resource and request another.
4. Circular wait is possible.

Deadlock Prevention:
Place restrictions on resource requests so that deadlock cannot occur
1. Allowing Preemption - 
	All resources can be preempted, but not possible practically.
2. Avoiding Mutual Exclusion - 
	Virtualize a resource using a resource scheduling process
	Create unlimtied virtual copies of the resource
	Useful only when we want to share a hardware resource instead of real-time access
3. Avoiding Hold & Wait -
	If a process can get hold of all the resources it needs at one time, no deadlock
	Inefficient
	Sometimes we don't know beforehand the resources that we would need
4. Avoiding Circular Wait -
	We can give each resource a unique, positive integer, and only acquire in an ascending order
	Inefficient, but often used

Deadlock Avoidance:
There are Algorithms for detecting Deadlock
Disadvantages:
	1. The algorithms are not fast
	2. Lot of overheads in running them before resource allocation
	3. Assumes every process know their max resources needs
	4. Assumes processes know what available resources are present

Deadlock Recovery:
1. Deadlock Detection - same as finding cycle in a graph, done periodically
2. Deadlock Resolution - Preempt an resource, and clear the block

Some Approaches to prevent Deadlock:
1. Advance Claim & Deadlock Avoidance algorithm
2. Always Allocate the requested resources

Starvation:
Process is prevented from proceeding because some other process is using the resource it needs
One solution is Aging - as a process waits, it ages and its priority increases (so oldest first scheduling)

Remote Procedure Calls (RPC)
We can various ways of process communication via messages:
	1. direct messaging to the processes with blocking send
	2. per-process message queues
	3. Blocking send-reply

In the third method, the sender will wait for the receiver to become ready to receive the message (blocked)
Then it will send the message and will wait for the receiver to send back a reply, after which both will proceed
This is common in client-server model
This is same as a procedure call (or a module call, where you send the arguments to a module call and wait for a return value)
This is the basis of remote procedure calls.
Defines the arguments and return value explicitly.

Here the messages are sent over a network, so it many not be in the same Address Space (address translation required and no references required, only complete values)
Also each of the stub function must know the network address of the other process.
Another problem is error handling.

Sender-Receiver have a subprocess/subfunctionality (known as a stub) which takes the procedure call request into a message over the network.
Example - Your request for a YouTube video is queried by the laptop NIC over the Internet to a server, which processes the request, and then sends a reply back via its own set of NICs.

RPC Algorithm:
1. Client makes a remote procedure call to another process's functionality.
2. Client-side stub will make it into a message on the network and send it.
3. Server-side stub will receive the message.
4. Server-side stub calls the real procedure on the server.
5. Real procedure performs the requested work and calls the server-side stub.
6. Server-stub sends the reply as a message.
7. Client-stub receives the message.
8. Client-stub returns to the remote procedure caller.

Monitors
Modules containing the following:
	1. Variables - any kind of data
	2. Condition Variables - Used for signalling inside the monitor
	3. Procedures - Can be called from outside the monitor

The variables' scope is within the monitor

The monitor ensures that only one procedure is called at one time (so a process sets a lock on the monitor)
Monitor provides Mutual Exclusion

Condition variables can only be defined within a monitor
They have two operations: wait & signal
They have no memory
If a process calls wait on a condition variable, it is made blocked until another process calls signals on it
wait also unlocks the monitor
If a process calls signal, it unblocks all waiting processes

MEMORY MANAGEMENT

There are 2 levels of memory management:
1. OS memory management - allocates large blocks of memory to processes
2. Process memory management - allocated a memory block from the OS, handles the internal management of it

In C++, process memory manager is 'new', and in C it is 'malloc'