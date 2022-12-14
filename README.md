
Multicasting using lamport's algorithm.

**Assignment description:**

Implement totally ordered multicasting using Lamport’s algorithm. Each process conducts local
operations and numbers them as PID.EVENT_ID. After each operation is done, a process
multicast the event to all other processes in the distributed system. The expected outcome of this
assignment is that events occurred at different processes will appear in the same order at each
individual process. To realize such a total order of events, each process maintains a buffer for
received events and follow the rules on slide 19 in Lecture 6 when delivering the events. In this
assignment, the delivery of events is simply printing them on screen, in the format of
CURRENT_PID: PID.EVENT_ID. HINT: You may use two threads in each process to handle
the communication and deal with message delivery, respectively. Refer to the multi-threaded
server example discussed in class. The communication thread enqueues updates in a buffer, from
where the delivery thread enforces a total order of events. The communication thread is also
responsible for sending acknowledgements for received messages.
Evaluation Standards
Correctness: Does the code's operation yield the desired outcome?
Parallelism: To expedite computing, does the parallel code function make advantage of many
GPU threads?
Required Tools:
For this project install the XTERM, it allows users to run programs which require a command line
interface. Install xterm by using the following command. Please change font size parameter as
required.

**Install Xterm using below command**
>> sudo apt -get - install xterm -y
Execution:
1. Here we implemented a totally ordered multicasting using Lamport’s algorithm. Each
process performs local activities and assigns them the EVENT_ID.
2. After completing each operation, then process broadcasts the event to all other processes
in the distributed system. This assignment is supposed to result in events occurring at
distinct processes appearing in the same order at each particular process.
3. To achieve such an event sequence, each process keeps a buffer for incoming events and
applies Lamport’s algorithm to order the events and then acknowledge by sending the
response to other processes. The events are simply printed on the screen
4. To execute the algorithm, write the code and save it with file name and an extension as
LamportMainProcess.java.
5. Create a bash script to execute the file using xterm script:
#! /bin/bash
xterm -fa 'Monospace' -fs 14 -e java LamportMainProcess.java 0 p0 &
xterm -fa 'Monospace' -fs 14 -e java LamportMainProcess.java 1 p1 &
xterm -fa 'Monospace' -fs 14 -e java LamportMainProcess.java 2 p2
6. Run the bash script using sh run.sh to execute.
7. The Process0 sends Lamport Event-0, Process1 sends Lamport Event-1 and Process2 sends
Lamport Event-2.
