Use this when you want to process multiple requests/functions concurrently

Think of this as telling Go: Go and do things in the background while i keep going
```
func main() {
	cook()
	eat()
}
Go does: cook -> finish -> eat
```

But with goroutine
```
func main() {
	go cook()
	eat()
}
it is now 
		| -> cook
main ->
		| -> eat
		
Saying; start cook. i don't want to wait till it finishes before calling eat
```

in same way, we can have something like
```
processTick(ticks []Tick) {
	for _, tick := range ticks {
		go func() {
			err := process(tick); ...
		}()
	}
}

func main() {
	ticks := []Tick{
		...
	}
	processTick(ticks)
	time.Sleep(5 * time.Seconds) // without this the processTick could finish before it's captured as they happen fast. To solve this, `waitgroup` comes to play
}
```
here, we have a loop housing a goroutine, meaning they'd execute at same time even though some delaying operations are going on within the process function

With the above, we don't really know when the goroutine finishes. to sync multiple goroutines and know when they finish, use WAITGROUP
### when data is involved, a channel comes into play
a channel is a pipe
`ch := make(chan string)`
`ch <- hello` you are putting hello into `ch`
to retrieve the value `value := <-ch`