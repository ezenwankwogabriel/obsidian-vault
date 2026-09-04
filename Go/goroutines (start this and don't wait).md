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
		-> cook
main ->
		-> eat
		
Saying; start cook. i don't want to wait till it finishes before calling eat
```

### when data is involved, a channel comes into play
a channel is a pipe
`ch := make(chan string)`
`ch <- hello` you are putting hello into `ch`
to retrieve the value `value := <-ch`