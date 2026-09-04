## basic idea
Time package deals with two diff concepts:
- Time -> what time/date is it
- Duration -> How long

## time.Time - a point in time
seeing `time.Time` -> a specific time for example `now := time.Now()` -> August 25, 2025, 5:20:12 PM
and can access methods like now.(Year/Month/Day/...)

### time.Duration - an amount of time
`5*time.Second` -> 5second
other examples are time.(Nanosecond/Millisecond/Microsecond/Hour/...)

### time.Sleep, time.After
time.Sleep(10 * time.Microsecond) -> pause a goroutine for 10microseconds

| Thing              | Think                                  |
| ------------------ | -------------------------------------- |
| `time.Time`        | **When?**                              |
| `time.Duration`    | **How long?**                          |
| `time.Now()`       | Current moment                         |
| `time.Sleep()`     | Pause current goroutine                |
| `time.After()`     | Channel that fires once after duration |
| `time.NewTimer()`  | Controllable one-shot timer            |
| `time.NewTicker()` | Repeated timer                         |
| `time.Since()`     | How long since a point                 |
| `t.Before()`       | Earlier?                               |
| `t.After()`        | Later?                                 |
| `t.Equal()`        | Same moment?                           |
```
TIME
 │
 ├── Time       → a moment
 │
 ├── Duration   → how long
 │
 ├── Timer      → do something once later
 │
 ├── Ticker     → do something repeatedly
 │
 └── After      → wait for a duration via a channel
```
