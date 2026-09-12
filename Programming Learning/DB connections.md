How many db connections should your application be allowed to open

Picture the app as a `restaurant` and `PostgreSQL` is the kitchen
- connection = a waiter currently connected to the kitchen
- max_open_connections = max no of waiters you're willing to have
- max_idle_connections = max no of waiters you're willing to keep around doing nothing
- max_idle_time = how long an idle waiter is allowed to stand idle before being sent home

A connection is needed for your app to be able to run queries
This connection isn't created from scratch every time, instead the app uses a connection pool.
```
                  ┌── Connection 1 ──┐
                  ├── Connection 2 ──┤
Application ──────┼── Connection 3 ──┼──── PostgreSQL
                  ├── Connection 4 ──┤
                  └── Connection 5 ──┘
                       Pool
```
This is because establishing a db connection has a cost. You wouldn't want such below
```
request
  ↓
create DB connection
  ↓
run query
  ↓
destroy DB connection
```
Instead we do:
```
Application starts
       ↓
Create some DB connections
       ↓
Keep them in pool
       ↓
Request arrives
       ↓
Borrow connection
       ↓
Run query
       ↓
Return connection to pool
```

### max open connections, 
`db.SetMaxOpenConns(20)`; the app can have at most 20 conns open
Once this is maxed out, requests needing a connection has to wait for one to become available.

### max idle connections 
`db.SetMaxIdleConns(5)` 
This means you have up to 20 connections open, but when not being used, keep at most 5 in the pool.
So if your app had 20 simultaneous requests
```
Request 1  → connection 1
Request 2  → connection 2
Request 3  → connection 3
...
Request 20 → connection 20
```
You temporarily have `20 open connections`

And if they finish, you don't need to destroy all 20 connections, instead only 5 idle is kept
```
               Connection Pool

IDLE:
  1
  2
  3
  4
  5

CLOSED:
  6
  7
  8
  ...
  20
```

Without keeping these idle connections, you'd have to initiate a new connection if a new request comes in instead of grabbing an existing one. This flow is much cheaper

### max idle time
db.SetConnMaxIdleTime(5 * time.Minutes)
This reads, keep idle connections around, but if a connection has been sitting unused for more than 5 minutes, close it.

```setup
db.SetMaxOpenConns(20)
db.SetMaxIdleConns(5)
db.SetConnMaxIdleTime(5 * time.Minute)
```

```max open
             MAX = 20

          ┌─────────────┐
          │ 20 possible │
          │ connections │
          └─────────────┘
```

```idle pool
After traffic dies down:

┌───┐ ┌───┐ ┌───┐ ┌───┐ ┌───┐
│ 1 │ │ 2 │ │ 3 │ │ 4 │ │ 5 │
└───┘ └───┘ └───┘ └───┘ └───┘
       idle connections

Only 5 are retained.
```

```
Idle for > 5 minutes
        ↓
     close it
```


## Where architecture matters
Suppose your psql has `max_connections = 100`
You shouldn't blindly configure `API = 100`. different instances, and workers would trigger this same amount of connections, maxing/overwhelming the instance

```
Postgres max_connections = 100

Reserve some for:
- administration
- migrations
- monitoring
- unexpected needs

Available to application ≈ 80

API:
  3 instances × 15 = 45

Workers:
  2 × 10 = 20

Other:
  15

Total ≈ 80
```
