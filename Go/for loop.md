
Both are the same `for` loop in Go, just different forms:

**`for condition { }`** — like a `while` loop. Condition is checked _before_ every iteration; the loop body never runs if it starts false.

```go
i := 0
for i < 10 {
    fmt.Println(i)
    i++
}
```

Use this when you have a simple boolean check that fully determines whether to keep looping, and you can evaluate it before doing any work.

**`for { ... }`** — infinite loop. You break out manually, usually with an `if` + `break` somewhere inside the body.

```go
for {
    line, err := reader.ReadString('\n')
    if err != nil {
        break
    }
    fmt.Println(line)
}
```

Use this when:

- The loop must do some work _before_ it can know whether to stop (e.g., reading from a stream — you don't know if there's more data until you try to read).
- The exit condition needs to be checked in the middle or at the end of the body, not the start.
- You need multiple different exit conditions (several `break`s, or a `return`) rather than one single boolean gate.

Rule of thumb: if you can write the stop condition as a value you already know _before_ entering the body, use `for condition {}`. If the condition only becomes knowable partway through the body's work, use `for { }` with a `break`.