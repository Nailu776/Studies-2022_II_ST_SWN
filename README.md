
# Token loss detection algorithm in a Token-Ring setting

## Authors

Julian Helwig - https://julian.helwig.tech/#/ https://github.com/Nailu776

Seweryn Kopeć - https://github.com/SewerynKopec

## Model:
- Channels have loss
- N number of processes
- One way communication in a ring
- There are no privilaged processes (all use the same code)
- Only a process with a valid token is able to access critical section
- The time for an unblocked message to propagate through the entire ring is known to all processes

## Communication:
There are 2 types of messages in a ring:
- Token - T(m), where m is the number of passes between processes last seen by the sender process  
- Acknowledge - (TTL,m), where TTL is Time To Live and m is the number of passes between processes last seen by the sender process 

## Description:  
The processes remember the number of passes of a token between them as local variable TPC (Token Passing Counter) in order to identify old irrelevant messages.

The processes send a token and after receiving one, they send a ACK(N-1,TPC)

If any messages with a different TPC value travel in the channels, they are ignored.  

If TTL > 1, the processes don't hold onto message, they pass it to their neighbor.

<div style="page-break-after: always;"></div>

## Concept image:
![Picture at Algorytm/example.png](Algorytm/example.png "Concept image")


## Detailed description:
### Variables:
- N - number of processes in the ring;
- n - process ID (any);
- TPC - Token Passing Counter - number of token passes know to the process during the sending;
- T, T(m) - token sent with TPC value equal to m;
- ACK, ACK(TTL,m) - acknowledge with Time To Live equal to TTL and TPC(from the sending process) equal to m;

<div style="page-break-after: always;"></div>

### Algorithm:
- The processes are only able to access the critical section when they are holding a token
- Each process constantly listens coming from the previous process in  the ring and holds a value TPC
- If proccess n:
    - Receives T(m) - it compares m and TPC:
        - m <= TPC - the token is outdated and ignored/removed
        - m > TPC - the token is valid, TPC is set to m+1
            - Process n stops listening for an ACK message, because it got back the token (it knows it reached the next process)
            - Process n sends a message ACK(N-1, TPC) to the next process in order to let the previous process know it received the token
            - Process n after exiting the critical section sends the token further as T(TPC) and begins to listen for the ACK 
    - Receives ACK(TTL,m) - it checks TTL and m and if:
        - TTL > 1 - it sends ACK(TTL-1,m) message
        - m > TPC - the message is valid, so it confirms the successful token transfer(For TTL > 1 it means that the message got lost between the process n and n+1 or it was slower than an ACK from the process n+k+1 to n+k, for any k>1, which also confirms a successful transfer)
    - Doesn't receive any confirmation until the time to go around the ring is elapsed
        - Retransmits the T(TPC) message
        - Starts to wait for a confirmation


