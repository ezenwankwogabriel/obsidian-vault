int => number without decimal places
float64 => number with decimal places
string => a text value
bool => true or false

uint => An unsigned integer meaning a strictly non-negative integer
int32 => A 32-bit signed integer meaning can range from -2,147,483,648 to 2,147,483,647
rune => An alias for int32, representing a unicode code point (a single character)
uint32 => A 32-bit unsigned integer: 0 - 4,294,967,295
int64 => A 64 signed integer

Null Values:
All Go value types come with a "null value", which is the value stored in a variable if no other value is explicitly set.