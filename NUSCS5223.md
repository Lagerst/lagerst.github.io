## Welcome

## [return to index](./index.html)

## Lecture 1 Remote Procedure Call

**RPC: Make a remote call look like a local function.**

> begin
> 
> client
> 
> 1.1 local call -> 1.2 pack arguments -> 1.3 transit then wait
> 
> server
> 
> 2.1 receive -> 2.2 unpack arguments -> 2.3 call, work
> 
> 2.4 return -> 2.5 pack result -> 2.6 transit
> 
> client
> 
> 1.4 receive -> 1.5 unpack result -> 1.6 local return
> 
> end;

**semantics:** 

**1** at-least-once RPC

> Client retries until response. But server have to ensure there are no side effect for dumplicated requests.

**2** at-most-once RPC

> Client includes a unique ID in every requests, server keeps a history of requests(ID, results) which already been executed. Server response with no re-execution.

**3** exact-once RPC

> At-most-once + client retries until success.

## Lecture 2 Time and Clocks

**Happens-before relationship - Lamport Clock**

Captures logical (causal) dependencies between events

• Within a process, a comes before b then a → b

• if a = send(M), and b = recv(M), then a → b

• (Irreflexive) Partial ordering: →

• a ❌→ a

• if a → b, then b ❌→ a

• (Transitivity) if a → b, and b → c, then a → c

• a ❌→ b and b ❌→ a: events are concurrent(no one can tell whether a or b happened first!)

**Logical clocks - Lamport Clock**

Implement a logical clock

• Keep a local clock T

• Increment T whenever an event happens

• Send clock value on all messages as Tm

• On message receipt: T = max(T, Tm) + 1


Use logical clock to form total ordering

• new relation: =>

• if C(a) < C(b), then a => b

• How about C(a) == C(b)?

• if a -> b, then a => b; but a => b doesn't mean a -> b;

**Issues about Lamport Clock**

• If a → b, then C(a) < C(b)

• if C(a) < C(b) then a → b?
> no, they could also be concurrent
> 
> if we were to use the Lamport clock as a global order, we would induce some unnecessary ordering constraints.

**Mutual exclusion**

• Use clocks to implement a lock

• Goals:
> Only one process has the lock at a time
> 
> Grant the lock in request order
> 
> Requesting processes eventually acquire the lock

• Assumptions:
> In-order point-to-point message delivery
> 
> No failures

**Mutual exclusion implementation**

• Each message carries a timestamp Tm

• Three message types:
> request (broadcast)
> 
> release (broadcast)
> 
> acknowledge (on receipt)

• Each node’s state:
> A queue of request messages, ordered by Tm
> 
> The latest timestamp it has received from each node

On receiving a request:
> Record message timestamp
> 
> Add request to queue
> 
> Send an acknowledgement

On receiving a release:
> Record message timestamp
> 
> Remove corresponding request from queue

On receiving an acknowledge:
> Record message timestamp

To acquire the lock:
> Send request to everyone, including self

The lock is acquired when:
> My request is at the head of my queue, and
> 
> I’ve received higher-timestamped messages from everyone
> 
> So my request must be the earliest

To release the lock:
> Send release to everyone, including self

**Vector clocks**

• Clock is a vector C, length = # of nodes

• On node i, increment C[i] on each event

• On receipt of message with clock Cm on node i

• Compare vectors element by element

• If Cx[i] < Cy[i] and Cx[j] > Cy[j] for some i, j, Cx and Cy are concurrent
> Example: (2, 3, 2) and (3, 0, 0)

• if Cx[i] <= Cy[i] for all i, and there exist j such that Cx[j] < Cy[j], Cx happens before Cy
> Example: (2, 3, 2) and (3, 0, 0)