# Rust Programming language

---

## Table of contents

1. [Common Programming Concepts](#common-programming-concepts)
    1. [Variables and Mutability](#variables-and-mutability)
    2. [Data Types](#data-types)
    3. [Functions](#functions)
    4. [Control Flow](#control-flow)
2. [Understanding Ownership](#understanding-ownership)
    1. [What Is Ownership?](#what-is-ownership)
    2. [References and Borrowing](#references-and-borrowing)

---

# Common Programming Concepts

## Variables and Mutability

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6; // will cause error since variables are not mutable by default
    println!("The value of x is: {x}");
}
```

```rust
// Everything is fine. Programm will be executed without any errors
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

### Constants

Like immutable variables, constants are values that are bound to a name and are not allowed to change, but there are a
few differences between constants and variables:

1. You aren’t allowed to use `mut` with constants. Constants aren’t just immutable by default—they’re always immutable.
2. You declare constants using the `const` keyword instead of the `let` keyword, and the type of the value must be
   annotated.
3. Constants can be declared in any scope, including the global scope, which makes them useful for values that many
   parts of code need to know about.
4. Constants may be set only to a constant expression, not the result of a value that could only be computed at runtime.

Rust’s naming convention for constants is to use all uppercase with underscores between words.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Constants are valid for the entire time a program runs, within the scope in which they were declared.

### Shadowing

You can declare a new variable with the same name as a previous variable. Rustaceans say that the first variable is
shadowed by the second, which means that the second variable is what the compiler will see when you use the name of the
variable.

In effect, the second variable overshadows the first, taking any uses of the variable name to itself until either it
itself is shadowed or the scope ends. We can shadow a variable by using the same variable’s name and repeating the use
of the let keyword as follows:

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```

## Data Types

### Scalar Types

- integers
- floating-point numbers
- booleans
- characters

### Compound Types

**Tuples**

A tuple is a general way of grouping together a number of values with a variety of types into one compound type.
Tuples have a fixed length: once declared, they cannot grow or shrink in size.

```rust
   fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);
    let five_hundred = x.0;
    let six_point_four = x.1;
    let one = x.2;
}
```

**Arrays**

Another way to have a collection of multiple values is with an array. Unlike a tuple, every element of an array must
have the same type. Unlike arrays in some other languages, arrays in Rust have a fixed length.

```rust
fn main() {
    let a: [i32; 5] = [1, 2, 3, 4, 5];
}
```

You can also initialize an array to contain the same value for each element by specifying the initial value, followed
by a semicolon, and then the length of the array in square brackets, as shown here:

```rust
fn main() {
    let a = [3; 5];
}
```

## Functions

We define a function in Rust by entering fn followed by a function name and a set of parentheses. The curly brackets
tell the compiler where the function body begins and ends.

We can call any function we’ve defined by entering its name followed by a set of parentheses. Because another_function
is defined in the program, it can be called from inside the main function. Note that we defined another_function after
the main function in the source code; we could have defined it before as well. Rust doesn’t care where you define your
functions, only that they’re defined somewhere in a scope that can be seen by the caller.

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

### Statements and Expressions

Function bodies are made up of a series of statements optionally ending in an expression. So far, the functions we’ve
covered haven’t included an ending expression, but you have seen an expression as part of a statement. Because Rust is
an expression-based language, this is an important distinction to understand. Other languages don’t have the same
distinctions, so let’s look at what statements and expressions are and how their differences affect the bodies of
functions.

- Statements are instructions that perform some action and do not return a value.
- Expressions evaluate to a resultant value.

Creating a variable and assigning a value to it with the `let` keyword is a statement. E.g. `let y = 6;` is a
statement.

Function definitions are also statements; the entire preceding example is a statement in itself.

Statements do not return values. Therefore, you can’t assign a let statement to another variable, as the following code
tries to do; you’ll get an error:

```
fn main() {
    let x = (let y = 6);
}
```

Expressions evaluate to a value and make up most of the rest of the code that you’ll write in Rust. Consider a math
operation, such as `5 + 6`, which is an expression that evaluates to the value `11`. Expressions can be part of
statements: the `6` in the statement `let y = 6`; is an expression that evaluates to the value `6`. Calling a function
is an expression. Calling a macro is an expression. A new scope block created with curly brackets is an expression, for
example:

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

Note that the `x + 1` line doesn’t have a semicolon at the end, which is unlike most of the lines you’ve seen so far.
Expressions do not include ending semicolons. If you add a semicolon to the end of an expression, you turn it into a
statement, and it will then not return a value.

### Functions with Return Values

Functions can return values to the code that calls them. We don’t name return values, but we must declare their type
after an arrow (`->`). In Rust, the return value of the function is synonymous with the value of the final expression in
the block of the body of a function. You can return early from a function by using the `return` keyword and specifying a
value, but most functions return the last expression implicitly.

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

## Control Flow

### Using `if` in a `let` Statement

Because `if` is an expression, we can use it on the right side of a `let` statement to assign the outcome to a variable:

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

### Repetition with Loops

The `loop` keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to
stop.

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

### Returning Values from Loops

One of the uses of a loop is to retry an operation you know might fail, such as checking whether a thread has completed
its job. You might also need to pass the result of that operation out of the loop to the rest of your code. To do this,
you can add the value you want returned after the break expression you use to stop the loop; that value will be returned
out of the loop so you can use it, as shown here:

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```

You can also `return` from inside a loop. While `break` only exits the current loop, `return` always exits the current
function.

### Loop Labels to Disambiguate Between Multiple Loops

If you have loops within loops, `break` and `continue` apply to the innermost loop at that point. You can optionally
specify a loop label on a loop that you can then use with `break` or `continue` to specify that those keywords apply to
the labeled loop instead of the innermost loop. Loop labels must begin with a single quote. Here’s an example with two
nested loops:

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

### Conditional Loops with `while`

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

### Looping Through a Collection with `for`

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```

Rust way to write the `fori` loop:

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```

---

# Understanding Ownership

## What Is Ownership?

Ownership is a set of rules that govern how a Rust program manages memory.

### Ownership Rules

- Each value in Rust has an owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Variable Scope

A scope is the range within a program for which an item is valid.

```rust
fn main() {
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
}
```

### Memory and Allocation

In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final
executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s
immutability. Unfortunately, we can’t put a blob of memory into the binary for each piece of text whose size is unknown
at compile time and whose size might change while running the program.

With the String type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on
the heap, unknown at compile time, to hold the contents. This means:

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we’re done with our `String`.

That first part is done by us: when we call `String::from`, its implementation requests the memory it needs. This is
pretty much universal in programming languages.

However, the second part is different. Rust takes a different from other languages path: the memory is automatically
returned once the variable that owns it goes out of scope. Here’s a version of our scope using a `String` instead of a
string literal:

```rust
fn main() {
    {
        let s = String::from("hello"); // s is valid from this point forward

        // do stuff with s
    } // this scope is now over, and s is no longer valid
}
```

There is a natural point at which we can return the memory our `String` needs to the allocator: when `s` goes out of
scope. When a variable goes out of scope, Rust calls a special function for us. This function is called `drop`, and it’s
where the author of `String` can put the code to return the memory. Rust calls `drop` automatically at the closing curly
bracket.

### Variables and Data Interacting with Move

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{s1}") // <-- error, because s1 is no longer valid
}
```

Earlier, we said that when a variable goes out of scope, Rust automatically calls the `drop` function and cleans up the
heap memory for that variable. This is a problem: when `s2` and `s1` go out of scope, they will both try to free the
same memory. This is known as a double free error and is one of the memory safety bugs we mentioned previously. Freeing
memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, after the line let `s2 = s1;`, Rust considers `s1` as no longer valid. Therefore, Rust doesn’t
need to free anything when `s1` goes out of scope.

If you’ve heard the terms shallow copy and deep copy while working with other languages, the concept of copying the
pointer, length, and capacity without copying the data probably sounds like making a shallow copy. But because Rust also
invalidates the first variable, instead of being called a shallow copy, it’s known as a move. In this example, we would
say that `s1` was moved into `s2`.

### Variables and Data Interacting with Clone

If we do want to deeply copy the heap data of the `String`, not just the stack data, we can use a common method
called `clone`.

```rust
fn main() {
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {s1}, s2 = {s2}"); // <-- everything works, no error here
}
```

When you see a call to `clone`, you know that some arbitrary code is being executed and that code may be expensive. It’s
a visual indicator that something different is going on.

### Stack-Only Data: Copy

Types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual
values are quick to make. That means there’s no reason we would want to prevent `x` from being valid after we create the
variable `let y = x`. In other words, there’s no difference between deep and shallow copying here, so calling `clone`
wouldn’t do anything different from the usual shallow copying, and we can leave it out.

Rust has a special annotation called the `Copy` trait that we can place on types that are stored on the stack, as
integers are. If a type implements the `Copy` trait, variables that use it do not move, but rather are trivially copied,
making them still valid after assignment to another variable.

Rust won’t let us annotate a type with `Copy` if the type, or any of its parts, has implemented the `Drop` trait. If the
type needs something special to happen when the value goes out of scope and we add the `Copy` annotation to that type,
we’ll get a compile-time error.

As a general rule, any group of simple scalar values can implement `Copy`, and nothing that requires allocation or is
some form of resource can implement `Copy`.

### Ownership and Functions

The mechanics of passing a value to a function are similar to those when assigning a value to a variable. Passing a
variable to a function will move or copy, just as assignment does.

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function, but i32 is Copy,... 
    // ... so it's okay to still use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{some_string}");
} // Here, some_string goes out of scope and `drop` is called. The backing memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{some_integer}");
} // Here, some_integer goes out of scope. Nothing special happens.
```

### Return Values and Scope

Returning values can also transfer ownership.

```rust
fn main() {
    let s1 = gives_ownership(); // gives_ownership moves its return value into s1

    let s2 = String::from("hello"); // s2 comes into scope

    let s3 = takes_and_gives_back(s2); // s2 is moved into takes_and_gives_back, which also moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String { // gives_ownership will move its return value into the function that calls it
    let some_string = String::from("yours"); // some_string comes into scope

    some_string // some_string is returned and moves out to the calling function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into scope
    a_string  // a_string is returned and moves out to the calling function
}
```

## References and Borrowing


