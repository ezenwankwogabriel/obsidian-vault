A distributed system: 11 service, diff languages, communicating primarily over gRPC.
Important Go services for us: frontend, productcatalog, checkout and shipping service.
```
                    Browser
                       │
                      HTTP
                       ↓
                  ┌─────────┐
                  │frontend │  Go
                  └────┬────┘
                       │ gRPC
                       ↓
                ┌──────────────┐
                │   checkout   │  Go
                └──────┬───────┘
                       │
             ┌─────────┼──────────┐
             ↓         ↓          ↓
          cart      payment    shipping
          C#         Node        Go
             │
           Redis
```

A normal Go application has
```
HTTP handler
   ↓
service
   ↓
repository
   ↓
database
```

Here, the boundary changes
```
HTTP handler
   ↓
Go function
   ↓
gRPC client
   ↓
──── NETWORK ────
   ↓
gRPC server
   ↓
Go function
```

Starting at the edge: "frontend" is the entry point. conceptually we have
```
Internet
   ↓
Kubernetes Service
   ↓
frontend Pod
   ↓
Go HTTP server
```

When entering any service, ask the question: *how does this process come alive*
For most, *main.go* is the answer

Suppose we have
```
Cart ✓
Payment ✓
Shipping ✗
```

we've already charged the customer, and now shipping fails. What should happen?

we can't simply do
```
payment.Rollback()
```
and assume everything is fine.

Payment might be another service, possibly another database, machine or the request that charged the card succeeded but its response was lost.

Important question, does every downstream operation need to wait for the previous one?