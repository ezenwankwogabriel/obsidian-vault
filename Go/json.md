### JSON
In Go, handling JSON revolves around encoding data into json bytes(`marshalling`) and decoding json bytes(`unmarshalling`)
##### Struct Tags: mapping rules
struct fields must be exported "started with a capital letter" for the json package to see them
>backticks are used to customize the json key names
```
type User struct {
	ID int `json:"id"`
	Name string `json:"full_name"`
	Email string `json:"email,omitempty"` // omit if empty
	Password string `json:"-"` // ignore completely
}
```

1. Decoding: Unmarshalling (json to go)
```
package main
import ("encoding/json" "fmt")

type User struct {
	Name string `json:"name"`
	Age int `json."age"`
}

func main() {
	jsonData := []byte(`{"name": "Alice", "age": 30}`)
	
	var user User
	err := json.Unmarshal(jsonData, &user)
	if err != nil {
		fmt.Println("Error decoding:", err)
	}
	
	fmt.Printf("%+v\n", user) // Output: {Name: Alice Age: ;30}
}
```

2. Encoding: Marshalling (go to json)
```
package main
import (encoding/json fmt)

type User struct {
	Name string `json:"name"`
	Age int `json."age"`
}

func main() {
	user := User{
		Name: "Alice",
		Age: 30
	}
	
	jsonData, err := json.Marshal(user)
	if err != nil {
		fmt.Println("Error encoding:", err)
		return
	}
	
	fmt.Println(string(jsonData)) // Output: {name: Bob, age: 25}
	
	
	var user User
	err := json.Unmarshal(jsonData, &user)
	if err != nil {
		fmt.Println("Error decoding:", err)
	}
	
	fmt.Printf("%+v\n", user) // Output: {Name: Alice Age: ;30}
} 
```

Working with HTTP Requests & Responses
for APi servers, use json.NewDecoder() and json.NewEncoder() to read/write directly from stream buffers.


Quick Cheat Sheet
Struct → JSON: json.Marshal(val) or json.NewEncoder(w).Encode(val)

JSON → Struct: json.Unmarshal(data, &val) or json.NewDecoder(r.Body).Decode(&val)

Dynamic / Unknown JSON: Decode into map[string]any when you don't have a fixed structure ahead of time.