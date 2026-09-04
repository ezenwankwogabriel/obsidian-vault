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

### With context, you can
- cancel (stop what you're doing)
- deadline/timeout (you have until this time)
- values (carry this request specific information)

### context.background()
ctx := context.Background() // used at top of application (root of work, having an empty context)
dowork(ctx)

### contexts form a tree
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