Think of this as a comms system ensuring two computers can reliably exchange data over a network

Allows your app send data to another app, with guarantees that the data arrives correctly, in the right order, and without missing pieces.

## **1. The problem TCP solves**

Imagine you're building a messaging application.

Alice sends:
> Hey, how are you doing?

Her message travels from her phone to your backend server.

But here's the problem: networks are not always reliable.

Several things can happen:

- Some data packets might get lost.
- Packets might arrive out of order.
- A packet might arrive twice.
- The receiving computer might be temporarily unavailable.
- The sender might not know whether the receiver got the data.

Without a mechanism to handle these problems, applications would have to implement their own reliability mechanisms.

**TCP solves this problem by providing reliable, ordered communication between two applications over an IP network.**

## **2. First, understand how data travels**

When your application sends data over a network, it doesn't necessarily send everything as one giant message.

TCP breaks the data into smaller units called **segments**.

For example, imagine sending this message:

```
Hello, welcome to our application!
```

TCP can divide it into smaller pieces:

```
Segment 1: Hello, 
Segment 2: welcome to 
Segment 3: our application!
```

These segments are then transmitted over the network.

At the receiving end, TCP reassembles the data into the original byte stream.

![[Pasted image 20261118131742.png]]

One important distinction:

- IP handles addressing and routing packets between computers.
- TCP handles reliable delivery and ordered data transmission between applications.

They work together.

## 3. **The TCP three-way handshake**

Before two applications communicate using TCP, they establish a connection.

This is called the **three-way handshake**.
![[Pasted image 20261118131942.png]]

It involves three steps:

**Step 1: SYN**

The client sends a SYN (synchronize) packet.

Essentially:

"Hey server, I'd like to establish a TCP connection."

**Step 2: SYN-ACK**

The server responds with SYN-ACK.

Essentially:

"I received your request. I'm ready to establish the connection too."

**Step 3: ACK**

The client sends an ACK.

Essentially:

"Great, let's communicate."

The connection is now established.

```
CLIENT                         SERVER
  |                               |
  |---------- SYN --------------->|
  |                               |
  |<-------- SYN-ACK -------------|
  |                               |
  |---------- ACK --------------->|
  |                               |
  |       CONNECTION ESTABLISHED   |
```

Why three steps?

Both sides need to confirm that they can send and receive data and synchronise their initial sequence numbers.

## 4. **TCP is connection-oriented**

This is a fundamental concept.

TCP is called a connection-oriented protocol because the two endpoints establish a logical connection before exchanging application data.

Consider a Node.js application:

```
import net from "node:net";

const client = net.createConnection({
  host: "example.com",
  port: 80,
});

client.on("connect", () => {
  console.log("TCP connection established");

  client.write("Hello server");
});

client.on("data", (data) => {
  console.log(data.toString());
});

client.on("error", (error) => {
  console.error(error);
});
```

Here, Node.js establishes a TCP connection to a server on port 80.

Once connected, it can send and receive data.

Notice that TCP itself doesn't understand HTTP, JSON, or your application's business logic.

It only understands bytes.

Your application protocol defines what those bytes mean.


Conceptually:

1. HTTP defines the request.
    
2. TCP provides reliable transport for HTTP/1.1 or HTTP/2.
    
3. IP routes packets to the destination.
    
4. Ethernet or Wi-Fi transmits the data across the local network.
    

HTTPS additionally uses TLS to encrypt the communication.


### practically
Imagine your frontend makes this request:

```
fetch("https://api.example.com/users");
```

Here's what happens:

**1. HTTP**

Creates the request:
`GET /users`
It defines what you want from the server.

HTTP data is passed down

**2. TCP**
Establishes a connection and reliably transports the HTTP data.

**3. IP**
Routes the packets to the destination.

**4. Network**
Sends the data through Wi-Fi, Ethernet, etc.

At the receiving end, the process is reversed. TCP reassembles the byte stream, and the HTTP implementation interprets it as an HTTP request.