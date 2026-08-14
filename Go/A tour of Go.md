- A name is exported if it begins with capital letter. `Pizza, Pi` as to `pizza and pi`
- inside a function (doesn't work outside a function), the := short assignment statement is used to place a var declaration with implicit type
- GO Types: (when you need an integer, always use `int` unless you have a specific reason to use a sized or unsigned integer type)
	- bool , string, int int8, int16, int21, int64 (and their uint), uintptr
	- byte // alias for uint8
	- rune // alias for int32 -> represents a Unicode code point
	- float32, float64
	- complex64, comples128
- T(v) converts the value to the type T.

Pointers:
A pointer holds the memory address of a value. type *T is a pointer to a T value
var p *int
The & operator generates a pointer to its operand: `i := 42 p = &i`

fmt.Println(*p) // read i through pointer p

type Vertex struct {
	X int
	Y int
}

func main() {
	fmt.Println(Vertex{1, 2})
}

An array has a fixed size meanwhile a slice has a dynamic size, flexible view into the elements of an array
Slice:
The type []T is a slice with elements of type T.





Things to read up on: 
- Slice Defaults



Methods in GO
- Go doesn't have classes, but you can define methods on types; just like in Rust

Pointer receivers (references in rust)
methods with pointers receivers can modify the value to which the receiver points

Methods and pointer inderection
var v Vertex
fmt.PRintln(AbsFunc(v))