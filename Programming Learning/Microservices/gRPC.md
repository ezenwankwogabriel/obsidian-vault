google RPC. This allows for RPC for clients & server in any language. They don't have to both be written in Go

// Find a protocol, compile your code, write the client code, write the server code and test


```
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative logs.proto
```
