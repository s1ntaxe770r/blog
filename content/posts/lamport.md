---
title: "Clocks, Event Ordering and Lamport"
date: 2024-10-11 16:16
draft: false
--- 

The order in which event's occur is an interesting topic and lately i have been thinking about it a lot. Particularly in the context of distributed systems.  

The rationale behind my thinking is fairly simple. In a large distributed system, how does the system maintain order ? Better still how does each process know which event came first? 

Before the rambling continues,  perhaps i should  start by defining what an *event* is.  An *event* simply refers an occurrence or action within a process. This could include sending or receiving a message or some kind of local action such as writing to disk.

## What happened to timestamps?
Time stamps are great and are easy to reason about. process $a$ and $b$ receive an event at time *t* . 
Determining which came first is a matter of checking which has the lower timestamp. 
Mathematically we can represent this as:

Let $t_a$ be the timestamp of event in process $a$
$$ t_a \text{ : timestamp of event in process } a $$
and $t_b$ be the timestamp of event in process $b$ $$ t_b \text{ : timestamp of event in process } b $$
If $t_a < t_b$ , event in process $a$ occurred first $$ t_a < t_b \implies \text{event in process } a \text{ occurred first} $$
we use the notation $a \rightarrow b$ to denote that event $a$ happens before event $b$. 

For example, if we want to say that an event in process $a$ happens before an event in process $b$, we can write: $$ t_a \rightarrow t_b $$
This  is read as "the event at time $t_a$ happens before the event at time $t_b$".

Easy enough,  but things start to fall apart we have to account for something called clock skew.  Clock skew occurs when the internal clocks of different computers are not perfectly synchronized.

Even in the same data center, two processes running on separate machines can experience clock skew due to factors such as clock crystals may ticking at slightly different rates,  latency in the NTP protocol.

That being said, order in distributed systems can  be divided in two type. Partial and total order. 

### Partial vs Total Order 
 The human mind views time as linear,  AKA event $a$  $\rightarrow$ $b$ $\rightarrow$ $c$  as time passes.  
 
 This is an easy way to think of  the concept of [total order](https://en.wikipedia.org/wiki/Total_order) where every element in a set is comparable or every element can be placed in a definite sequence ,  therefore we can say the system can be totally ordered. 
![lineartime](lamport/linear-time.png)
As you might have guess partial ordering is the opposite(kind of). Mathematically,  it can be defined as a set in which some pairs of events, we can determine their order, but for others, we cannot.  

In a distributed system, events $a$ and $b$ on different processes might be concurrent ($a || b$), meaning we can't determine which happened first.
## What about Leslie? 
In 1978 [Leslie Lamport](https://www.google.com/search?client=safari&rls=en&q=leslie+lamport&ie=UTF-8&oe=UTF-8) wrote a paper titled "Time, Clocks, and the Ordering of Events in a Distributed System",  he proposed  the concept of logical clocks(lamport clocks) , which provide a way to assign timestamps to events in a distributed system without relying on physical clocks.

Part of why this post even exists is because I needed an excuse to implement a Lamport clock, which we will get to shortly.

## Implementing  a Lamport Clock
I highly doubt my implementation is great but reading and implementing concepts from and academic paper was a fun exercise. 

I began by creating an enum to represent of the three possible events the paper outlines:
```go
type Event int

const (
    Send = iota
    Received
    Local
)
```

### Defining a Clock

Next, I created a structure to represent a Lamport clock:
```go
type LamportClock struct {
    counter atomic.Int32
    mutext  sync.Mutex
}
```
using `atomic.Int32` i can somewhat guarantee thread safety , the counter here is also important because the paper states:

>we define a clock Ci for each process Pi
to be a function which assigns a number Ci(a) to any
event a in that process. 

The mutex is an implementation detail to ensure thread-safety. 

Now, let's look at the methods:
```go
func(lc *LamportClock)Tick(currentClock int32) {
    currentTime := max(lc.counter.Load(), currentClock) + 1
    lc.counter.Store(currentTime)
}

func(lc *LamportClock) Local() {
    lc.counter.Add(1)
}

func(lc *LamportClock)CurrentTimestamp() int32 {
 return lc.counter.Load()
}
```

The `Tick` method implements the core of Lamport's clock synchronization. When a message is received, the clock is updated to be greater than both its current value and the timestamp of the received message. 

`max(lc.counter.Load(), currentClock)`  compares the local clock value with the received timestamp (`currentClock`). We need to use `max` here to ensure that the new clock value is greater than both

The local clock value (to maintain the local process order) and  the received timestamp (to respect the "happens-before" relationship with the sending process)


The `Local` method increments the counter for local events. As Lamport put it: 

> Each process $P_i$ increments Ci between any two successive events.

The `CurrentTimestamp` method simply returns the current value of the logical clock, which can be used when sending messages to other processes.

### Processes 
Here i tried to model individual processes or services as nodes in a distributed system. 

Each node has its own Lamport clock and can send and receive messages. 

```go
type Node interface {
    Send(CurrentTimestamp int32) int32
    Receive(event clock.Event, timestamp int32)
}
```

I also needed to define a `Message` struct to represent the messages exchanged between nodes:
```go
type Message struct {
    SenderID  string
    Timestamp int32
    Content   string
}
```
Each message contains the sender's ID, the Lamport timestamp, and the message content.

I used a `Service`  struct to represent a process with an embedded Lamport clock:
```go
type Service struct {
    Name        string
    Id          string
    logger      *slog.Logger
    MessageChan chan Message
    Clock       clock.LamportClock
    mutext      sync.RWMutex
}
```

In order for the `Service` struct to implement the `node` interface i added three methods 

**Send Method**:
```go
func (s *Service) Send(CurrentTimestamp int32) int32 {
    s.Clock.Local()
    msg := Message{
        SenderID:  s.Id,
        Timestamp: s.Clock.CurrentTimestamp(),
        Content:   "HI LESLIE!!!",
    }
    s.mutext.Lock()
    s.MessageChan <- msg
    s.mutext.Unlock()
    s.logger.Info("sent message", "Timestamp", msg.Timestamp, "Content", msg.Content)
    return s.Clock.CurrentTimestamp()
}
```
This method implements the sending of a message. It increments the local clock, creates a message with the current timestamp, and sends it through the message channel. 

Per the paper :
> "A process increments its counter before each event in that process."

Similarly the Recieve Method:
```go
func (s *Service) Receive(event clock.Event, timestamp int32) {
    s.Clock.Tick(timestamp)
}
```

And finally a helper function for handling received messages 
```go
func (s *Service) HandleMessages() {
    s.logger.Info("Starting message handler")
    for msg := range s.MessageChan {
        s.Receive(clock.Received, msg.Timestamp)
        s.logger.Info("Received Message", "from", msg.SenderID, "timestamp", msg.Timestamp, "Content", msg.Content)
    }
}
```

### Validating my Implementation
Testing stuff like this is weird, there isn't exactly a guide titled "testing your distributed system". But it seemed sane to write some tests to ensure the implementation worked somewhat. 

```go

func TestSingleTick(t *testing.T) {
	messageChan := make(chan node.Message, 100)
	OrderService := node.NewService("OrderService", messageChan)

	OrderService.Receive(clock.Received, OrderService.Clock.CurrentTimestamp())

	assert.Equal(t, 1, int(OrderService.Clock.CurrentTimestamp()))

}

func TestSend(t *testing.T) {
	messageChan := make(chan node.Message, 100)

	OrderService := node.NewService("OrderService", messageChan)
	go OrderService.HandleMessages()

	PaymentService := node.NewService("PaymentService", messageChan)
	CurrentTimestamp := PaymentService.Clock.CurrentTimestamp()
	PaymentService.Send(CurrentTimestamp)

	// Give some time for the message to be processed
	time.Sleep(100 * time.Millisecond)

	assert.Equal(t, 2, int(OrderService.Clock.CurrentTimestamp()))
}
```
## Tick , [Tick](https://www.youtube.com/watch?v=tQAF5W6NS88)...
while my doubts about my implementation still remain, I learnt a ton about more about clocks and shockingly the human perception of time. Beyond that  it was great to get a way from the grasp of Kubernetes. 

The full implementation is available [here](https://github.com/s1ntaxe770r/whyport-this-clock) , also a lot of the information here wouldn't be possible without these amazing authors: 

- [Christian Galatolo](https://medium.com/outreach-prague/lamport-clocks-determining-the-order-of-events-in-distributed-systems-41a9a8489177)
- [Lamport Clocks by Fredrico Ponzi](https://blog.fponzi.me/2024-02-02-lamport-clocks.html#what-problem-are-they-trying-to-solve)
- [ The Original Paper](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)
- [Time and Order by](https://book.mixu.net/distsys/time.html) [Mikito Takada](http://mixu.net/).






