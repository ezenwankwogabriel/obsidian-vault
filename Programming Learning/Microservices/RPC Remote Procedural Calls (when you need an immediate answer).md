*caveat: application in client & server must be written in go*

For communications between microservices
Http is still maintained for user-browser calls

this works for go for only applications written in "go". i.e a js microservice would not connect to a go microservice using rpc

PROs
- It is faster than marshalling and unmarshalling JSON strings



rpc.Register() >> tells go what services are available
rpcListen() tells go where to listen for clients
rpc.ServeConn() tells go how to handle each connected clients

GOAL: Exposing an RPC service
an application could say: `hey log-service, call loginfo with this data`
and the log-service will
```
receive RPC request
		↓
RPCServer.LogInfo(...)
		↓
write to MongoDB
		↓
return response
```

RPC service
```
type RPCServer struct {
	client *mongo.Client
}

func (r *RPCServer) LogInfo(payload RPCPayload, resp *string) error {
	...
}
```

From the above > I have a service called RPCServer, and it provides a method called LogInfo
A remote service should eventually be able to say something like "RPCService.LogInfo"

rpc.Register(new(RPCServer))
This doesn't start the server. It simply tells Go's RPC package
`Hey, when an RPC request comes in asking for RPC service, RPCServer is one of the services I offer`

```
rpc.Register(new(RPCServer)) 
is essentially
server := new(RPCServer)
rpc.Register(server)

this is morelike initializing the RPCServer `RPCServer{}`
```

app.rpcListen()
This starts the network listener
```
func (app *Config) rpcListen() error {
    log.Println("starting RPC server on port: ", rpcPort)

    listen, err := net.Listen(
        "tcp",
        fmt.Sprintf("0.0.0.0:%s", rpcPort),
    )
```
this is where port "5001" gets opened

So .register says "this is what i can do (methods)"
While rpcListen says "here's where you can connect to me"

rpc.ServeConn(rpcConn) means > "now handle that customers request"
This is where the magic happens, it reads the RPC protocol coming across that TCP connection
If client intends to call RPCServer.LogInfo, GO looks at the service registered earlier `rpc.Register(new(RPCServer))`
and see if it has RPCServer.LogInfo

complete flow

```
                    YOUR LOG SERVICE
                    ────────────────

main()
  │
  │
  ├── rpc.Register(new(RPCServer))
  │       │
  │       └──── tells Go:
  │             "RPCServer.LogInfo exists"
  │
  │
  ├── go app.rpcListen()
  │       │
  │       └── net.Listen(":5001")
  │               │
  │               ▼
  │           PORT 5001
  │           LISTENING
  │               │
  │               │
  │               │ client connects
  │               ▼
  │          listen.Accept()
  │               │
  │               ▼
  │            rpcConn
  │               │
  │               ▼
  │        go rpc.ServeConn()
  │               │
  │               │ reads RPC request
  │               ▼
  │        "RPCServer.LogInfo"
  │               │
  │               ▼
  │        RPCServer.LogInfo()
  │               │
  │               ▼
  │            MongoDB
```