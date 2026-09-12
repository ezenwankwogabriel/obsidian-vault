
HTTP was initially created for hypertext documents (documents with links to other documents)
Then developers found HTTP could send images & videos too. now is used for APIs, file transfers, ...
Here, a new TCP connection is created for each request/response exchange btw clients and server. All requests incur a latency penalty because of this.

HTTP 1.1 introduced a "keep-alive" connection to solve the problem, allowing clients to reuse TCP connections. This amortizes (reduces) the cost of establishing a TCP connection. Multiple requests could share the same connection. 
But still, they had to be serialized one after the other, so a client & server could only execute a single request/response exchange at any given time for each connection.

HTTP 2 introduced the concept of streams. An abstraction allowing HTTP to concurrently multiplex different HTTP exchanges to same TCP connection (allowing browsers to reuse TCP connections). Still, there was concern about packet loss as (e.g due to network congestion) because TCP does not know this abstraction, seeing it all as a stream of bytes with no particular meaning.
![[Pasted image 20260906160344.png]]

HTTP 3 used quic instead of TCP. quic was based built on UDP (connectionless). It introduces streams as first-class citizens at the transport layer. QUIC streams share same QUIC connection, so no additional handshake/slow starts are required to create new one




HTTP 2 depends on multiplexing requests on single connections as opposed to new connections for every requests on HTTP 1.1. This added advantage introduces an issue of packet loss during transmission using TCP which results in a resend of the whole request as TCP enforces ordered delivery. All packets after the lost one are held in buffer until the gap is filled (example: a css file arriving fine gets blocked because packet #3 from an unrelated image was dropped) 
HTTP 3 fixes this by introducing streams, it uses QUIC (a new internet protocol) based off UDP datagram and merges TLS 1.3 directly into its transport handshake.