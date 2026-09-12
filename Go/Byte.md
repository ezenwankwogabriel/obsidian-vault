A small box that can hold a number from 0 - 255 (2^8)
this is an alias for uint8
`var a byte = 64` and `var b uint8 = 64` is same thing

A byte has 8 bits
```
1 byte
┌────────────────────────┐
│ 0 1 0 0 0 0 0 1        │
└────────────────────────┘
   8 bits
```

Why do we use bytes so much in Go?
This is because computer fundamentals deal with data as sequences of bytes.
```
data := []bytes("hello)

this converts the string into bytes
h → 104
e → 101
l → 108
l → 108
o → 111

fmt.Println(data) => [104 101 108 108 111]
fmt.Println(string(data)) => hello
```

A practical example: HTTP
When you read a request body `body, err := io.ReadAll(resp.Body)`
body is `[]byte`
then you can convert that to string `fmt.Println(string(body))`
or JSON-decode the bytes directly `json.Unmarshal(body, &user)`

### byte vs character
A byte isn't necessarily a character. 
```
for simple ascii
A = 1 byte
B = 1 byte
hello = 5 bytes

but unicode characters can require multiple bytes
len("😊") => 4
```

## String vs Bytes
A string is a read-only sequence of bytes.
```
fmt.Println([]byte(name)) => [71 97 98 114 105 101 108]

the numbers are the UTF-8 bytes for => G  a  b  r  i  e  l
```

### Why have both
Sometimes you're dealing with Bytes and other times with Strings.
When reading a file like a jpg, you're dealing with byte data as the data isn't text, it's raw binary data
you don't want string(data) cause jpg can't be interpreted as a text

String is immutable. you cannot do
```
name := "hello"
name[0] = 'J' // ❌

// instead can have
data := []byte("hello")
data[0] = "j"
fmt.Println(string(data)) => "jello"
```