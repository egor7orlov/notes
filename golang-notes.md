# Go Programming Language

---

## Types

Go’s types fall into four categories:

- basic types
- aggregate types
- reference types
- interface types

## Basic types

### Integers

- int8 / uint8
- int16 / uint16
- int32 / uint32
- int64 / uint64
- int / uint *(size depends on compiler; usually 32 or 64 bits)*
- rune - synonym for int32 and conventionally indicates that a value is a Unicode codepoint

### Floating-Point Numbers

- float32 - approximately six decimal digits of precision
- float64 - about 15 digits

### Complex Numbers

> TODO: investigate what complex numbers are, because I haven't understood anything from the book

### Booleans

- true
- false

### Strings

A string is an immutable sequence of bytes.

Text strings are conventionally interpreted as UTF-8-encoded sequences of Unicode code points (runes).

#### Substring operation

The substring operation `s[i:j]` yields a new string consisting of the bytes of the original string starting at index i
and continuing up to, but not including, the byte at index j. The result contains j-i bytes. Either or both of the i and
j operands may be omitted, in which case the default values of 0 (the start of the string) and `len(s)` (its end) are
assumed, respectively.

#### Strings' immutability explanation

Immutability means that it is safe for two copies of a string to share the same underlying memory, making it cheap to
copy strings of any length. Similarly, a string s and a substring like `s[7:]` may safely share the same data, so the
substring operation is also cheap. No new memory is allocated in either case.

#### String Literals

Because Go source files are always encoded in UTF-8 and Go text strings are conventionally interpreted as UTF-8, we can
include Unicode code points in string literals.

A raw string literal is written \`...\`, using backquotes instead of double quotes. Within a raw string literal, no
escape sequences are processed; the contents are taken literally, including backslashes and newlines, so a raw string
literal may spread over several lines in the program source. The only processing is that carriage returns are deleted so
that the value of the string is the same on all platforms, including those that conventionally put carriage returns in
text files.

#### UTF-8

Converting an integer value to a string interprets the integer as a rune value, and yields the UTF-8 representation of
that rune:

```
fmt.Println(string(65)) // "A", not "65" 
fmt.Println(string(0x4eac)) // "C"
```

If the rune is invalid, the replacement character is substituted:

```
fmt.Println(string(1234567)) // "�"
```

### Constants

Constants are expressions whose value is known to the compiler and whose evaluation is guaranteed to occur at compile
time, not at run time. The underlying type of every constant is a basic type: boolean, string, or number.

---

## Composite Types

Arrays and structs are aggregate types; their values are concatenations of other values in memory. Arrays are
homogeneous — their elements all have the same type—whereas structs are heterogeneous.

Both arrays and structs are fixed size. In contrast, slices and maps are dynamic data structures that grow as values are
added.

### Arrays

An array is a fixed-length sequence of zero or more elements of a particular type.

By default, the elements of a new array variable are initially set to the zero value for the element type, which is 0
for numbers. We can use an array literal to initialize an array with a list of values:

```
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}

fmt.Println(r[2]) // "0"
```

In an array literal, if an ellipsis ‘‘...’’ appears in place of the length, the array length is determined by the number
of initializers.

```
q := [...]int{1, 2, 3}

fmt.Printf("%T\n", q) // "[3]int"
```

The size of an array is part of its type, so `[3]int` and `[4]int` are different types. The size must be a constant
expression, that is, an expression whose value can be computed as the program is being compiled.

If an array’s element type is comparable then the array type is comparable too, so we may directly compare two arrays of
that type using the == operator, which reports whether all corresponding elements are equal.

```
a := [2]int{1, 2}
b := [...]int{1, 2}
c := [2]int{1, 3}

fmt.Println(a == b, a == c, b == c) // "true false false"

d := [3]int{1, 2}

fmt.Println(a == d) // compile error: cannot compare [2]int == [3]int
```

### Slices

Slices represent variable-length sequences whose elements all have the same type. A slice type is written `[]T`, where
the elements have type `T`; it looks like an array type without a size.

A slice is a lightweight data structure that gives access to a subsequence (or perhaps all) of the elements of an array,
which is known as the slice’s underlying array.

A slice has three components:

- pointer
- length
- capacity.

The pointer points to the first element of the array that is reachable through the slice, which is not necessarily the
array’s first element. The length is the number of slice elements; it can’t exceed the capacity, which is usually the
number of elements between the start of the slice and the end of the underlying array. The built-in functions `len`
and `cap` return those values.

Multiple slices can share the same underlying array and may refer to overlapping parts of that array.

The slice operator `s[i:j]`, where **0 ≤ i ≤ j ≤ `cap(s)`**, creates a new slice that refers to elements i through j-1
of the sequence s, which may be an array variable, a pointer to an array, or another slice. The resulting slice has j-i
elements. If i is omitted, it’s 0, and if j is omitted, it’s `len(s)`.

> Slicing beyond `cap(s)` causes a panic, but slicing beyond `len(s)` extends the slice

Since a slice contains a pointer to an element of an array, passing a slice to a function permits the function to modify
the underlying array elements. In other words, copying a slice creates an *alias* for the underlying array.

> Each time we take the address of a variable or copy a pointer, we create new aliases or ways to identify the same
> variable. For example, *p is an alias for v:
>
> ```
> func incr(p *int) int {
>     *p++ // increments what p points to; does not change p
>     return *p
> }
> 
> v := 1
> 
> incr(&v)              // side effect: v is now 2
> fmt.Println(incr(&v)) // "3" (and v is 3)
> ```

A slice literal looks like an array literal, a sequence of values separated by commas and surrounded by braces, but the
size is not given. This implicitly creates an array variable of the right size and yields a slice that points to it.

As with array literals, slice literals may specify the values in order, or give their indices explicitly, or use a mix
of the two styles.

Unlike arrays, slices are not comparable, so we cannot use == to test whether two slices contain the same elements. The
standard library provides the highly optimized bytes.Equal function for comparing two slices of bytes (`[]byte`), but
for other types of slice, we must do the comparison ourselves.

### Maps

Map is an unordered collection of key/value pairs in which all the keys are distinct, and the value associated with a
given key can be retrieved, updated, or removed using a constant number of key comparisons on the average, no matter how
large the hash table.

In Go, a map is a reference to a hash table, and a map type is written `map[K]V`, where K and V are the types of its
keys and values.

The key type K must be comparable using `==`, so that the map can test whether a given key is equal to one already
within it. Though floating-point numbers are comparable, it’s a bad idea to compare floats for equality and especially
bad if NaN is a possible value. There are no restrictions on the value type V.

Accessing a map element by subscripting always yields a value. If the key is present in the map, you get the
corresponding value; if not, you get the zero value for the element type. For many purposes that’s fine, but sometimes
you need to know whether the element was really there or not. For example, if the element type is numeric, you might
have to distinguish between a nonexistent element and an element that happens to have the value zero, using a test like
this:

```
age, ok := ages["bob"]

if !ok { /* "bob" is not a key in this map; age == 0. */ }
```

You’ll often see these two statements combined, like this:

```
if age, ok := ages["bob"]; !ok { /* ... */ }
```

### Structs

A struct is an aggregate data type that groups together zero or more named values of arbitrary types as a single entity.
Each value is called a field.

The zero value for a struct is composed of the zero values of each of its fields. It is usually desirable that the zero
value be a natural or sensible default.

#### Comparing structs

If all the fields of a struct are comparable, the struct itself is comparable, so two expressions of that type may be
compared using == or !=. The == operation compares the corresponding fields of the two structs in order, so the two
printed expressions below are equivalent:

```
type Point struct{ X, Y int }

p := Point{1, 2}
q := Point{2, 1}

fmt.Println(p.X == q.X && p.Y == q.Y) // "false"
fmt.Println(p == q) // "false"
```

Comparable struct types, like other comparable types, may be used as the key type of a map.

```
type address struct {
    hostname string
    port int
}

hits := make(map[address]int)
hits[address{"golang.org", 443}]++
```

#### Struct Embedding and Anonymous Fields

Go lets us declare a field with a type but no name; such fields are called anonymous fields. The type of the field must
be a named type or a pointer to a named type. Below, Circle and Wheel have one anonymous field each. We say that a Point
is embedded within Circle, and a Circle is embedded within Wheel.

```
type Point struct {
    X, Y int
}

type Circle struct {
    Point
    Radius int
}

type Wheel struct {
    Circle
    Spokes int
}

var w Wheel
w.X = 8         // equivalent to w.Circle.Point.X = 8
w.Y = 8         // equivalent to w.Circle.Point.Y = 8
w.Radius = 5 // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```

Unfortunately, there’s no corresponding shorthand for the struct literal syntax, so neither of these will compile:

```
w = Wheel{8, 8, 5, 20}                       // compile error: unknown fields
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
```

---

## Functions

A function declaration has a name, a list of parameters, an optional list of results, and a body:

```
func name(parameter-list) (result-list) { body
}
```

The type of a function is sometimes called its signature. Two functions have the same type or signature if they have the
same sequence of parameter types and the same sequence of result types.

### Error-Handling Strategies

- Error propagation

```
resp, err := http.Get(url)
if err != nil {
    return nil, err
}

doc, err := html.Parse(resp.Body)
resp.Body.Close()
if err != nil {
    return nil, fmt.Errorf("parsing %s as HTML: %v", url, err)
}
```

- Retry

### Variadic Functions

A variadic function is one that can be called with varying numbers of arguments. The most familiar examples
are `fmt.Printf` and its variants.

```
func sum(vals ...int) int {
    total := 0
    
    for _, val := range vals {
        total += val
    }

    return total
}

// Normal usage
fmt.Println(sum()) //  "0"
fmt.Println(sum(3)) //  "3"
fmt.Println(sum(1, 2, 3, 4)) //  "10"

// Usage with arguments alreday collected in slice
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

Although the ...int parameter behaves like a slice within the function body, the type of a variadic function is distinct
from the type of a function with an ordinary slice parameter.

```
func f(...int) {}
func g([]int)  {}

fmt.Printf("%T\n", f) // "func(...int)"
fmt.Printf("%T\n", g) // "func([]int)"
```

### Deferred Function Calls

Syntactically, a defer statement is an ordinary function or method call prefixed by the keyword defer. The function and
argument expressions are evaluated when the statement is executed, but the actual call is deferred until the function
that contains the defer statement has finished, whether normally, by executing a return statement or falling off the
end, or abnormally, by panicking. Any number of calls may be deferred; they are executed in the reverse of the order in
which they were deferred.

```
// Working wit mutexes

var mu sync.Mutex
var m = make(map[string]int)

func lookup(key string) int {
    mu.Lock()
    defer mu.Unlock()
    return m[key]
}

// Modifying result right before returning it to the caller

func triple(x int) (result int) {
    defer func () { result += x }()
    return double(x)
}


fmt.Println(triple(4)) // "12"
```

### Panic

During a typical panic, normal execution stops, all deferred function calls in that goroutine are executed, and the
program crashes with a log message. This log message includes the panic value, which is usually an error message of some
sort, and, for each goroutine, a stack trace showing the stack of function calls that were active at the time of the
panic.

If the built-in `recover` function is called within a deferred function and the function containing the defer statement
is panicking, recover ends the current state of panic and returns the panic value. The function that was panicking does
not continue where it left off but returns normally. If recover is called at any other time, it has no effect and
returns nil.

```
func Parse(input string) (s *Syntax, err error) {
    defer func() {
        if p := recover(); p != nil {
            err = fmt.Errorf("internal error: %v", p)
        }
    }()
    // ...parser...
}
```

---

## Methods

### Method Declarations

```
type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

The extra parameter `p` is called the method’s receiver, a legacy from early object-oriented languages that described
calling a method as "sending a message to an object".

Since methods and fields inhabit the same name space, declaring a method `X` on the struct type `Point` would be
ambiguous and the compiler will reject it.

#### Methods with a Pointer Receiver

Because calling a function makes a copy of each argument value, if a function needs to update a variable, or if an
argument is so large that we wish to avoid copying it, we must pass the address of the variable using a pointer. The
same goes for methods that need to update the receiver variable: we attach them to the pointer type, such as `*Point`.

```
func (p *Point) ScaleBy(factor float64) {
    p.X *= factor
    p.Y *= factor
}
```

> In a realistic program, convention dictates that if any method of a type has a pointer receiver, then all methods of a
> type should have a pointer receiver, even ones that don’t strictly need it.

Named types and pointers to them are the only types that may appear in a receiver declaration. Furthermore, to avoid
ambiguities, method declarations are not permitted on named types that are themselves pointer types:

```
type P *int
func (P) f() { /* ... */ } // compile error: invalid receiver type
```

### Composing Types by Struct Embedding

```
type Point struct{ X, Y float64 }

func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

type ColoredPoint struct {
    Point
    Color color.RGBA
}

// ...

var cp ColoredPoint
cp.X = 1
fmt.Println(cp.Point.X) // "1"
cp.Point.Y = 2
fmt.Println(cp.Y) // "2"
```

### Method Values and Expressions

The selector `p.Distance` yields a ***method value***, a function that binds a method to a specific receiver value `p`.
This function can then be invoked without a receiver value; it needs only the non-receiver arguments.

```
p := Point{1, 2}
q := Point{4, 6}
distanceFromP := p.Distance     // method value
fmt.Println(distanceFromP(q))   // "5"
```

Related to the method value is the ***method expression***. When calling a method, as opposed to an ordinary function,
we must supply the receiver in a special way using the selector syntax. A method expression, written `T.f` or `(*T).f`
where `T` is a type, yields a function value with a regular first parameter taking the place of the receiver, so it can
be called in the usual way.

```
p := Point{1, 2}
q := Point{4, 6}
distance := Point.Distance   // method expression
fmt.Println(distance(p, q))  // "5"
fmt.Printf("%T\n", distance) // "func(Point, Point) float64"
```
