
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

SAMPLE CODE:


package assignment1;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.net.InetAddress;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.*;

public class LamportMainProcess {
    public static void main(String  [] args) throws IOException, InterruptedException {
        LamportMultiCastAlgorithm lamport = new LamportMultiCastAlgorithm();
        String param1 = args[0];
        String param2 = args[1];
        lamport.startProcess(param1, param2);
    }
}

class LamportEvent implements Serializable {

    public LamportEvent(String processIdString, int processId, int logicalTime, String eventId, boolean acknowledgement) {
        this.processIdString = processIdString;
        this.processId = processId;
        this.logicalTime = logicalTime;
        this.eventId = eventId;
        this.acknowledgement = acknowledgement;
    }

    private int processId;

    private String processIdString;
    private int logicalTime;
    private String eventId;
    private boolean acknowledgement;

    public String getProcessIdString() {
        return processIdString;
    }

    public int getProcessId() {
        return processId;
    }

    public int getLogicalTime() {
        return logicalTime;
    }

    public void setLogicalTime(int logicalTime) {
        this.logicalTime = logicalTime;
    }

    public String getEventId() {
        return eventId;
    }

    public boolean isAcknowledged() {
        return acknowledgement;
    }

}

class LamportMultiCastAlgorithm {
    public static final String COLOUR_MODE_OFF = "\u001B[0m";
    public static final String RED_MODE = "\u001B[41m";
    public static final String GREEN_MODE = "\u001B[42m";
    public static final String YELLOW_MODE = "\u001B[43m";
    public static String currentColour ;
    private static ServerSocket serverSocket;
    public static String processIdString;
    public static int processIdIdentifier;
    private static int eventsProcessed = 0;
    private static int logicalTimeCounter = 0;
    public static int[] portList = new int[]{8001,8002,8003};
    public static String[] eventList = new String[]{"LamportEvent-0", "LamportEvent-1", "LamportEvent-2"};

    public static volatile Map<String, Set<String>> acknowledgementBuffer = new HashMap<>();
    public static volatile List<LamportEvent> lamportEventBuffer = new ArrayList<>();

    // Lamport MultiCastAlgorithm total order algorithm
    public static Comparator<LamportEvent> customComparator = (a, b) -> (
            (a.getLogicalTime() != b.getLogicalTime()
                    && a.getLogicalTime() > b.getLogicalTime())
                    || (a.getProcessId() > b.getProcessId())) ? 1 : -1;

    // LamportMulti CastAlgorithm clock incrementer
    public synchronized void incrementLogicalTimeCounter(LamportEvent e){
        // we need to increment logic time counter for all process incoming from other process
        // so eliminating current system process as it already time stamped
        if(e.getProcessId() != processIdIdentifier) {
            logicalTimeCounter = Math.max(logicalTimeCounter,e.getLogicalTime())+1;
            e.setLogicalTime(logicalTimeCounter);
        }

    }

    public void startProcess(String param1, String param2) throws IOException, InterruptedException {
        processIdIdentifier = Integer.parseInt(param1);
        processIdString = param2;
        // start the thread which will manage communications
        (new Thread(() -> {
            try {
                manageCommunications();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        })).start();

        // we are sleeping the process for 1 second, so in the mean time
        // all the other process will br ready to accept the requests
        // this is to simulate multi threaded environment on vm
        Thread.sleep(1000);

        // start the thread to order events
        (new Thread(() -> deliveredEvents())).start();


        // start the thread to manage acknowledgements
        (new Thread(() -> {
            try {
                manageAcknowledgements();
            } catch (IOException e) {
                throw new RuntimeException(e);
            }
        })).start();


        // multicast 2 events from each service. This can be extended to n services
        if(processIdIdentifier==0) {
            multicastEvent(new LamportEvent(processIdString, processIdIdentifier, logicalTimeCounter, eventList[0], false));
        } else if (processIdIdentifier==1){
            multicastEvent(new LamportEvent(processIdString, processIdIdentifier, logicalTimeCounter, eventList[1], false));
        }
        else if (processIdIdentifier==2){
            multicastEvent(new LamportEvent(processIdString, processIdIdentifier, logicalTimeCounter, eventList[2], false));
        }
    }


    private void manageAcknowledgements() throws IOException {
        while (eventsProcessed != eventList.length) {
            if (!lamportEventBuffer.isEmpty()) {
                LamportEvent lamportEvent = lamportEventBuffer.get(0);
                if (canAcknowledge(lamportEvent)) {
                    incrementLogicalTimeCounter(lamportEvent);
                    multicastEvent(new LamportEvent(processIdString, processIdIdentifier, logicalTimeCounter, lamportEvent.getEventId(), true));

                    if (acknowledgementBuffer.containsKey(lamportEvent.getEventId())) {
                        acknowledgementBuffer.get(lamportEvent.getEventId()).add(processIdString);
                    } else {
                        Set<String> set = new HashSet<>() {{
                            add(processIdString);
                        }};
                        acknowledgementBuffer.put(lamportEvent.getEventId(), set);
                    }
                }
            }
        }
    }

    private boolean canAcknowledge(LamportEvent lamportEvent) {
        // check if lamportEvent is already acknowledged
        if(!acknowledgementBuffer.isEmpty()
                && acknowledgementBuffer.containsKey(lamportEvent.getEventId())
                && acknowledgementBuffer.get(lamportEvent.getEventId()).contains(processIdString)){
            return false;
        }

        // check if its lamportEvent from current process and if it already acknowledged
        if((lamportEvent.getProcessId() == processIdIdentifier)
                || ((acknowledgementBuffer.containsKey(eventList[processIdIdentifier])
                && acknowledgementBuffer.get(eventList[processIdIdentifier]).size() == portList.length))) return true;


        // smaller process id will have higher precedence
        if((logicalTimeCounter == lamportEvent.getLogicalTime()
                && processIdIdentifier > lamportEvent.getProcessId())
                ||logicalTimeCounter > lamportEvent.getLogicalTime()){
            return true;
        }

        return false;
    }

    }
    }
    }


