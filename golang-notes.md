# Go Programming Language

---

Notes based on the contents of ["The Go Programming Language" book](https://www.amazon.com/Programming-Language-Addison-Wesley-Professional-Computing/dp/0134190440).

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

## Interfaces

An interface is an *abstract type*. It doesn’t expose the representation or internal structure of its values, or the set
of basic operations they support; it reveals only some of their methods. When you have a value of an interface type, you
know nothing about what it is; you know only what it can do, or more precisely, what behaviors are provided by its
methods.

```
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Closer interface {
    Close() error
}
```

*Embedding* an interface:

```
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```

The type `interface{}`, which is called the ***empty interface*** type, is indispensable. Because the empty interface
type places no demands on the types that satisfy it, we can assign any value to the empty interface.

---

## Goroutines and Channels

Go enables two styles of concurrent programming:

- ***Communicating sequential processes*** or ***CSP***, a model of concurrency in which values are passed between
  independent activities (goroutines) but variables are for the most part confined to a single activity.
- More traditional model of ***shared memory multithreading***, which will be familiar if you’ve used threads in other
  mainstream languages.

### Goroutines

In Go, each concurrently executing activity is called a ***goroutine***. When a program starts, its only goroutine is
the one that calls the main function, so we call it the *main goroutine*.

New goroutines are created by the `go` statement. Syntactically, a `go` statement is an ordinary function or method call
prefixed by the keyword `go`. A go statement causes the function to be called in a newly created goroutine. The `go`
statement itself completes immediately:

```
f()    // call f(); wait for it to return
go f() // create a new goroutine that calls f(); don't wait
```

### Channels

If goroutines are the activities of a concurrent Go program, channels are the connections between them. A channel is a
communication mechanism that lets one goroutine send values to another goroutine. Each channel is a conduit for values
of a particular type, called the channel’s element type. The type of a channel whose elements have type int is written
chan int.
To create a channel, we use the built-in make function:

```
ch := make(chan int) // ch has type 'chan int'
```

As with maps, a channel is a reference to the data structure created by make. When we copy a channel or pass one as an
argument to a function, we are copying a reference, so caller and callee refer to the same data structure. As with other
reference types, the zero value of a channel is nil.

A channel has two principal operations, *send* and *receive*, collectively known as *communications*.

```
ch <- x   // a send statement
x = <-ch  // a receive expression in an assignment statement
<-ch      // a receive statement; result is discarded
close(ch) // closing channel; attempts to send anything to it will cause panic 
```

#### Unbuffered Channels

A send operation on an unbuffered channel blocks the sending goroutine until another goroutine executes a corresponding
receive on the same channel, at which point the value is transmitted and both goroutines may continue. Conversely, if
the receive operation was attempted first, the receiving goroutine is blocked until another goroutine performs a send on
the same channel.

Communication over an unbuffered channel causes the sending and receiving goroutines to synchronize. Because of this,
unbuffered channels are sometimes called synchronous channels.

#### Unidirectional Channel Types

The Go type system provides unidirectional channel types that expose only one or the other of the send and receive
operations.

- The type `chan<- int`, a send-only channel of int, allows sends but not receives.
- The type `<-chan int`, a receive-only channel of int, allows receives but not sends.

Violations of this discipline are detected at compile time.

Since the `close` operation asserts that no more sends will occur on a channel, only the sending goroutine is in a
position to call it, and for this reason it is a compile-time error to attempt to `close` a receive-only channel.

#### Buffered Channels

A buffered channel has a queue of elements. The queue’s maximum size is determined when it is created, by the capacity
argument to `make`.

```
ch = make(chan string, 3)
```

A send operation on a buffered channel inserts an element at the back of the queue, and a receive operation removes an
element from the front. If the channel is full, the send operation blocks its goroutine until space is made available by
another goroutine’s receive. Conversely, if the channel is empty, a receive operation blocks until a value is sent by
another goroutine.

### Multiplexing with select

General form of `select` statement:

```
select {
     case <-ch1:
         // ...
     case x := <-ch2:
         // ...use x...
     case ch3 <- y:
         // ...
     default:
}
```

Example:

```
ch := make(chan int, 1)
     for i := 0; i < 10; i++ {
         select {
         case x := <-ch:
             fmt.Println(x) // "0" "2" "4" "6" "8"
         case ch <- i:
    }
}
```

If multiple cases are ready, select picks one at random, which ensures that every channel has an equal chance of being
selected.

---

## Concurrency with Shared Variables

### Race Conditions

In a program with two or more goroutines, the steps within each goroutine happen in the familiar order, but in general
we don’t know whether an event x in one goroutine happens before an event y in another goroutine, or happens after it,
or is simultaneous with it. When we cannot confidently say that one event happens before the other, then the events x
and y are *concurrent*.

A ***race condition*** is a situation in which the program does not give the correct result for some interleavings of
the operations of multiple goroutines. Race conditions are pernicious because they may remain latent in a program and
appear infrequently, perhaps only under heavy load or when using certain compilers, platforms, or architectures.

A *data race* occurs whenever two goroutines access the same variable concurrently and at least one of the accesses is a
write.

### Mutual Exclusion: sync.Mutex

The *mutex* prevents the shared variables from being simultaneously accessed by multiple goroutines.

```
import "sync"

var (
    mu      sync.Mutex
    balance int
)

func Deposit(amount int) {
    mu.Lock()
    balance = balance + amount
    mu.Unlock()
}

func Balance() int {
    mu.Lock()
    b := balance
    mu.Unlock()
    return b
}
```

The region of code between Lock and Unlock in which a goroutine is free to read and modify the shared variables is
called a ***critical section***.

> By convention, the variables guarded by a mutex are declared immediately after the declaration of the mutex itself. If
> you deviate from this, be sure to document it.

When dealing with more complex logic it's better to release mutex using `defer` statement. The Unlock executes after the
return statement has read the value of balance, so the Balance function is concurrency-safe.

```
func Balance() int {
    mu.Lock()
    defer mu.Unlock()
    
    // ... Super complex logic  with branching and error handling goes here 
    
    return balance
}
```

### Read/Write Mutexes: sync.RWMutex

Since the Balance function only needs to read the state of the variable, it would in fact be safe for multiple Balance
calls to run concurrently, so long as no Deposit or Withdraw call is running. In this scenario we need a special kind
of lock that allows read-only operations to proceed in parallel with each other, but write operations to have fully
exclusive access. This lock is called a *multiple readers, single writer* lock, and in Go it’s provided by sync.RWMutex:

```
var mu sync.RWMutex
var balance int

func Balance() int {
    mu.RLock() // readers lock
    defer mu.RUnlock()
    return balance
}
```

The Balance function now calls the RLock and RUnlock methods to acquire and release a readers or shared lock. The
Deposit function, which is unchanged, calls the mu.Lock and mu.Unlock methods to acquire and release a writer or
exclusive lock.

### Memory Synchronization

There are two reasons we need a mutex. The first is that it’s equally important that Balance not execute in the middle
of some other operation like Withdraw. The second (and more subtle) reason is that synchronization is about more than
just the order of execution of multiple goroutines; synchronization also affects memory.

In a modern computer there may be dozens of processors, each with its own local cache of the main memory. For
efficiency, writes to memory are buffered within each processor and flushed out to main memory only when necessary. They
may even be committed to main memory in a different order than they were written by the writing goroutine.
Synchronization primitives like channel communications and mutex operations cause the processor to flush out and commit
all its accumulated writes so that the effects of goroutine execution up to that point are guaranteed to be visible to
goroutines running on other processors.

### Goroutines and Threads

#### Growable Stacks

Each OS thread has a fixed-size block of memory (often as large as 2MB) for its stack, the work area where it saves the
local variables of function calls that are in progress or temporarily suspended while another function is called.

In contrast, a goroutine starts life with a small stack, typically 2KB. A goroutine’s stack, like the stack of an OS
thread, holds the local variables of active and suspended function calls, but unlike an OS thread, a goroutine’s stack
is not fixed; it grows and shrinks as needed.

#### Goroutine Scheduling

OS threads are scheduled by the OS kernel. Every few milliseconds, a hardware timer interrupts the processor, which
causes a kernel function called the scheduler to be invoked. This function suspends the currently executing thread and
saves its registers in memory, looks over the list of threads and decides which one should run next, restores that
thread’s registers from memory, then resumes the execution of that thread. Because OS threads are scheduled by the
kernel, passing control from one thread to another requires a full context switch, that is, saving the state of one user
thread to memory, restoring the state of another, and updating the scheduler’s data structures. This operation is slow,
due to its poor locality and the number of memory accesses required, and has historically only gotten worse as the
number of CPU cycles required to access memory has increased.

The Go runtime contains its own scheduler that uses a technique known as m:n scheduling, because it multiplexes (or
schedules) m goroutines on n OS threads. The job of the Go scheduler is analogous to that of the kernel scheduler, but
it is concerned only with the goroutines of a single Go program. Unlike the operating system’s thread scheduler, the Go
scheduler is not invoked periodically by a hardware timer, but implicitly by certain Go language constructs.

#### GOMAXPROCS

The Go scheduler uses a parameter called GOMAXPROCS to determine how many OS threads may be actively executing Go code
simultaneously. Its default value is the number of CPUs on the machine, so on a machine with 8 CPUs, the scheduler will
schedule Go code on up to 8 OS threads at once. (GOMAXPROCS is the n in m:n scheduling.) Goroutines that are sleeping or
blocked in a communication do not need a thread at all. Goroutines that are blocked in I/O or other system calls or are
calling non-Go functions, do need an OS thread, but GOMAXPROCS need not account for them.

#### Goroutines Have No Identity

In most operating systems and programming languages that support multithreading, the current thread has a distinct
identity that can be easily obtained as an ordinary value, typically an integer or pointer. This makes it easy to build
an abstraction called thread-local storage, which is essentially a global map keyed by thread identity, so that each
thread can store and retrieve values independent of other threads.

Goroutines have no notion of identity that is accessible to the programmer. This is by design, since thread-local
storage tends to be abused.

---

## Packages and the Go Tool

### Import Paths

Each package is identified by a unique string called its import path. Import paths are the strings that appear in import
declarations:

```
import (
    "fmt"
    "math/rand"
    "encoding/json"
    "golang.org/x/net/html"
    "github.com/go-sql-driver/mysql"
)
```

### The Package Declaration

A package declaration is required at the start of every Go source file. Its main purpose is to determine the default
identifier for that package (called the package name) when it is imported by another package.

For example, every file of the math/rand package starts with package rand, so when you import this package, you can
access its members as rand.Int, rand.Float64, and so on.

```
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println(rand.Int())
}
```

Conventionally, the package name is the last segment of the import path, and as a result, two packages may have the same
name even though their import paths necessarily differ.

There are three major exceptions to the "last segment" convention:

- The first is that a package defining a command (an executable Go program) always has the name main, regardless of the
  package’s import path.
- The second exception is that some files in the directory may have the suffix _test on their package name if the file
  name ends with _test.go. Such a directory may define two packages: the usual one, plus another one called an external
  test package.
- The third exception is that some tools for dependency management append version number suffixes to package import
  paths, such as "gopkg.in/yaml.v2". The package name excludes the suffix, so in this case it would be just yaml.

### Import Declarations

A Go source file may contain zero or more import declarations immediately after the package declaration and before the
first non-import declaration.

If we need to import two packages whose names are the same, like math/rand and crypto/rand, into a third package, the
import declaration must specify an alternative name for at least one of them to avoid a conflict. This is called a
renaming import.

```
import (
    "crypto/rand"
    mrand "math/rand" // alternative name mrand avoids conflict
)
```

The alternative name affects only the importing file. Other files, even ones in the same package, may import the package
using its default name, or a different name.

### Blank Imports

To suppress the "unused import" error we would otherwise encounter, we must use a renaming import in which the
alternative name is _, the blank identifier. As usual, the blank identifier can never be referenced.

```
import (
    "database/sql"
    _ "github.com/lib/pq"              // enable support for Postgres
    _ "github.com/go-sql-driver/mysql" // enable support for MySQL
)

// ...

db, err = sql.Open("postgres", dbname) // OK
db, err = sql.Open("mysql", dbname)    // OK
db, err = sql.Open("sqlite3", dbname)  // returns error: unknown driver "sqlite3"
```

---

## Testing

### The `go test` Tool

The go test subcommand is a test driver for Go packages that are organized according to certain conventions. In a
package directory, files whose names end with _test.go are not part of the package ordinarily built by go build but are
a part of it when built by go test.

Within *_test.go files, three kinds of functions are treated specially: tests, benchmarks, and examples.

- A test function, which is a function whose name begins with Test, exercises some program logic for correct behavior;
  go test calls the test function and reports the result, which is either PASS or FAIL.
- A benchmark function has a name beginning with Benchmark and measures the performance of some operation; go test
  reports the mean execution time of the operation.
- An example function, whose name starts with Example, provides machine-checked documentation.

### Test Functions

Each test file must import the testing package. Test functions have the following signature:

```
func TestName(t *testing.T) {
  // ...
}
```

Test function names must begin with Test; the optional suffix Name must begin with a capital letter:

```
func TestSin(t *testing.T) { /* ... */ }
func TestCos(t *testing.T) { /* ... */ }
func TestLog(t *testing.T) { /* ... */ }
```

### Benchmark Functions

a benchmark function looks like a test function, but with the Benchmark prefix and a *testing.B parameter that provides
most of the same methods as a *testing.T, plus a few extra related to performance measurement. It also exposes an
integer field N, which specifies the number of times to perform the operation being measured.

```
import "testing"

func BenchmarkIsPalindrome(b *testing.B) {
    for i := 0; i < b.N; i++ {
        IsPalindrome("A man, a plan, a canal: Panama")
    }
}
```

Unlike tests, by default no benchmarks are run. The argument to the -bench flag selects which benchmarks to run. It is a
regular expression matching the names of Benchmark functions, with a default value that matches none of them. The ‘‘.’’
pattern causes it to match all benchmarks.

```
$ go test -bench=.
```

### Profiling

When we wish to look carefully at the speed of our programs, the best technique for identifying the critical code is
profiling. Profiling is an automated approach to performance measurement based on sampling a number of profile events
during execution, then extrapolating from them during a post-processing step; the resulting statistical summary is
called a profile.

- A ***CPU profile*** identifies the functions whose execution requires the most CPU time. The currently running thread
  on each CPU is interrupted periodically by the operating system every few milliseconds, with each interruption
  recording one profile event before normal execution resumes.
- A ***heap profile*** identifies the statements responsible for allocating the most memory. The profiling library
  samples calls to the internal memory allocation routines so that on average, one profile event is recorded per 512KB
  of allocated memory.
- A ***blocking profile*** identifies the operations responsible for blocking goroutines the longest, such as system
  calls, channel sends and receives, and acquisitions of locks. The profiling library records an event every time a
  goroutine is blocked by one of these operations.

Gathering a profile for code under test is as easy as enabling one of the flags below. Be careful when using more than
one flag at a time, however: the machinery for gathering one kind of profile may skew the results of others.

```
$ go test -cpuprofile=cpu.out
$ go test -blockprofile=block.out
$ go test -memprofile=mem.out
```

Once we’ve gathered a profile, we need to analyze it using the `pprof` tool. This is a standard part of the Go
distribution, but since it’s not an everyday tool, it’s accessed indirectly using `go tool pprof`. It has dozens of
features and options, but basic use requires only two arguments, the executable that produced the profile and the
profile log.

> `go test` usually discards the test executable once the test is complete, when profiling is enabled it saves the
> executable as "foo.test", where foo is the name of the tested package.

The commands below show how to gather and display a simple CPU profile. We’ve selected one of the benchmarks from
the `net/http` package. It is usually better to profile specific benchmarks that have been constructed to be
representative of workloads one cares about. Benchmarking test cases is almost never representative, which is why we
disabled them by using the filter `-run=NONE`.

```
$ go test -run=NONE -bench=ClientServerParallelTLS64 -cpuprofile=cpu.log net/http
$ go tool pprof -text -nodecount=10 ./http.test cpu.log
```

### Example Functions

The third kind of function treated specially by go test is an example function, one whose name starts with Example. It
has neither parameters nor results. Here’s an example function for `IsPalindrome`:

```
func ExampleIsPalindrome() {
    fmt.Println(IsPalindrome("A man, a plan, a canal: Panama"))
    fmt.Println(IsPalindrome("palindrome"))
    // Output:
    // true
    // false
}
```

Example functions serve three purposes:

- Documentation: a good example can be a more succinct or intuitive way to convey the behavior of a library function
  than its prose description, especially when used as a reminder or quick reference. Based on the suffix of the Example
  function, the web-based documentation server godoc associates example functions with the function or package they
  exemplify, so ExampleIsPalindrome would be shown with the documentation for the IsPalindrome function, and an example
  function called just Example would be associated with the word package as a whole.
- Examples are executable tests run by go test. If the example function contains a final `// Output:` comment like the
  one above, the test driver will execute the function and check that what it printed to its standard output matches the
  text within the comment.
- The third purpose of an example is hands-on experimentation. The godoc server at golang.org uses the Go Playground to
  let the user edit and run each example function from within a web browser.

---

## Reflection
