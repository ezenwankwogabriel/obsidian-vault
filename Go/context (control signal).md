[allows us cancel long running operations]
Think of this as a control signal that travels with a piece of work
Imagine you make an HTTP request
```
User
 │
 ▼
HTTP handler
 │
 ├── Database query
 │
 ├── Call payment service
 │
 └── Call email service
```
Imagine the user closes the browser during this process.
You don't want the server to keep doing all the unnecessary work. instead you say the operation is canceled, everyone stop working.

## What is a context
It represents a lifetime, cancellation, deadline, and request-scoped values of some work.
`think of this as a little object travelling down your call chain`
everyone gets same context
```
func Handler(ctx context.Context) {
    Service(ctx)
}

func Service(ctx context.Context) {
    Repository(ctx)
}

func Repository(ctx context.Context) {
    // use ctx
}
```
Without context, you may need to define some other way to stop a running action. maybe with a channel, passing it around. with context used as a D.I, you can have a canceling mechanism.

### Things to remember
- cancellation - stop work when the request is canceled
- deadline/timeout - stop work if it takes too long
- request-scoped values - carry things like request IDs through the call chain

### Basic Idea
imagine `Http request -> Handler -> Service -> Repository -> PostgreSQL`
The HTTP request might disappear before the db operation finishes

```
User requests /users/123
        ↓
Handler
        ↓
Service
        ↓
Database query ────────────────┐
                               │
User closes browser            │
        ↓                      │
Request cancelled              │
                               ↓
                         DB operation
                         should stop
```
`context.Context` allows cancellation signal to travel down the chain

#### Example 1
```
func GetUser(ctx context.Context, id int) (User, error) {
    return repository.FindUser(ctx, id)
}

func FindUser(ctx context.Context, id int) (User, error) {
    // database operation using ctx
}
```
With the above, if the request is canceled -> Ctx is caneled -> service knows -> repository knows and -> DB can stop

### Example 2
Suppose your API should never spend more than 2 seconds processing something
```
ctx, cancel := context.withTimeout(
	context.Background(),
	2*time.Second,
)
defer cancel()

user, err := GetUser(ctx, 123)
```
if GetUser takes longer than 2 seconds, context canceled, operations using ctx should stop
```
select {
	case <-ctx.Done() // check if the requirement is reached (2 seconds)
		return ctx.Err()
	default:
}
```

### HTTP servers gives you a context
```
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    // use ctx
}
```
No need to create one here, if the client disconnects, the request context is cancelled.

```
# http handler
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    user, err := getUser(ctx, 123)

    if err != nil {
        http.Error(w, err.Error(), 500)
        return
    }

    // respond
}

# then the getUser func
func getUser(ctx context.Context, id int) (User, error) {
    return repo.findUser(ctx, id)
}

# and the actual repository call
func (repo *Repo) findUser(ctx context.Context, id int) (User, error) {
    return repo.db.QueryContext(
        ctx,
        "SELECT id, name FROM users WHERE id = $1",
        id,
    )
}
```
You'd notice same ctx is passed round. Suppose postgresql is taking forever:
client disconnects -> http context canceled -> ctx.Done() => QueryContext notices cancellation -> DB operation can terminate.

## One major rule
Done create `context.Background` deep inside your application.

### context.background()
ctx := context.Background() // used at top of application (root of work, having an empty context)
dowork(ctx)

### contexts from a tree
parent := context.Background()
child := context.WithCancel(parent)

### ctx.Done
to know when a context is done and wants to cancel


## Important functions
`context.Background` // root context
`ctx, cancel := context.WithCancel(parent)` // manual cancellation (stop when i say so)
`ctx, cancel := context.WithTimeout(parent, 6*time.Second)` // automatic cancellation after a duration
`ctx, cancel := context.WithDeadline(parent, deadline)` // cancel auto at specific time
`<-ctx.Done()` // has the context been cancelled?

```
func main() {
	ctx, cancel := context.WithTimeout(
		context.Background(),
		5*time.Second()
	)
	defer cancel()
	
	resultCh := make(chan string)
	
	go func() {
		time.Sleep(3*time.Second())
		resultCh <- "finished"
	}()
	
	select {
	case result := <-resultCh:
		fmt.Println(result)
	case <-ctx.Done()
		fmt.Println("operation stopped", ctx.Err())
	}
}
```