## Raft-based Distributed key-value store

### High-Level Architecture

Imagine you have three independent server instances running on your machine (Nodes A, B, and C).

- **Clients** send standard HTTP requests (`PUT`, `GET`, `DELETE`) to store and retrieve data.
    
- **Nodes** talk to each other under the hood using **gRPC** and the **Raft consensus algorithm** to ensure that all three nodes agree on the exact same sequence of data changes, even if one or two nodes suddenly crash or lose network connectivity.