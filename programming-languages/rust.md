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
    3. [The Slice Type](#the-slice-type)
3. [Using Structs to Structure Related Data](#using-structs-to-structure-related-data)
    1. [Defining and Instantiating Structs](#defining-and-instantiating-structs)
    2. [Method Syntax](#method-syntax)
4. [Enums and Pattern Matching](#enums-and-pattern-matching)
    1. [Defining an Enum](#defining-an-enum)
    2. [The `match` Control Flow Construct](#the-match-control-flow-construct)
    3. [Concise Control Flow with if let](#concise-control-flow-with-if-let)
5. [Managing Growing Projects with Packages, Crates, and Modules](#managing-growing-projects-with-packages-crates-and-modules)
    1. [Defining Modules to Control Scope and Privacy](#defining-modules-to-control-scope-and-privacy)
6. [Common Collections](#common-collections)
    1. [Storing Lists of Values with Vectors](#storing-lists-of-values-with-vectors)
    2. [Storing UTF-8 Encoded Text with Strings](#storing-utf-8-encoded-text-with-strings)
    3. [Storing Keys with Associated Values in Hash Maps](#storing-keys-with-associated-values-in-hash-maps)
7. [Error Handling](#error-handling)
    1. [Unrecoverable Errors with `panic!`](#unrecoverable-errors-with-panic)
    2. [Recoverable Errors with `Result`](#recoverable-errors-with-result)
8. [Generic Types, Traits, and Lifetimes](#generic-types-traits-and-lifetimes)
    1. [Traits: Defining Shared Behavior](#traits-defining-shared-behavior)
    2. [Validating References with Lifetimes](#validating-references-with-lifetimes)
9. [Writing Automated Tests](#writing-automated-tests)
    1. [How to Write Tests](#how-to-write-tests)
    2. [Controlling How Tests Are Run](#controlling-how-tests-are-run)
    3. [Test Organization](#test-organization)

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

Here is how you would define and use a `calculate_length` function that has a reference to an object as a parameter
instead of taking ownership of the value:

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{s1}' is {len}.");
}

fn calculate_length(s: &String) -> usize { // `s` is a reference to a String
    s.len()
} // Here, `s` goes out of scope. But because it does not have ownership of what it refers to, it is not dropped.
```

Note that we pass `&s1` into `calculate_length` and, in its definition, we take `&String` rather than `String`. These
ampersands represent _references_, and they allow you to refer to some value without taking ownership of it.

> Note: The opposite of referencing by using & is dereferencing, which is accomplished with the dereference
> operator, `*`.

We call the action of creating a reference _borrowing_. As in real life, if a person owns something, you can borrow it
from them. When you’re done, you have to give it back. You don’t own it.

### Mutable References

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Mutable references have one big restriction: if you have a mutable reference to a value, you can have no other
references to that value. This code that attempts to create two mutable references to `s` will fail:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s; // <-- error: second mutable borrow occurs here

    println!("{}, {}", r1, r2);
}
```

The restriction preventing multiple mutable references to the same data at the same time allows for mutation but in a
very controlled fashion. The benefit of having this restriction is that Rust can prevent data races at compile time.

As always, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not
simultaneous ones:

```rust
fn main() {
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
}
```

Rust enforces a similar rule for combining mutable and immutable references. This code results in an error:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    let r3 = &mut s; // BIG PROBLEM

    println!("{}, {}, and {}", r1, r2, r3);
}
```

We also cannot have a mutable reference while we have an immutable one to the same value. Users of an immutable
reference don’t expect the value to suddenly change out from under them! However, multiple immutable references are
allowed because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is
used. For instance, this code will compile because the last usage of the immutable references, the `println!`, occurs
before the mutable reference is introduced:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{r1} and {r2}");
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{r3}");
}
```

### The Rules of References

- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.

## The Slice Type

_Slices_ let you reference a contiguous sequence of elements in a collection rather than the whole collection. A slice
is a kind of reference, so it does not have ownership.

### String Slices

A _string slice_ is a reference to part of a `String`, and it looks like this:

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
}
```

Recall that we talked about string literals being stored inside the binary. Now that we know about slices, we can
properly understand string literals:

```rust
fn main() {
    let s = "Hello, world!";
}
```

The type of `s` here is `&str`: it’s a slice pointing to that specific point of the binary. This is also why string
literals are immutable; `&str` is an immutable reference.

### Other Slices

Just as we might want to refer to part of a string, we might want to refer to part of an array. We’d do so like this:

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let slice = &a[1..3];

    assert_eq!(slice, &[2, 3]);
}
```

This slice has the type `&[i32]`. It works the same way as string slices do, by storing a reference to the first element
and a length. You’ll use this kind of slice for all sorts of other collections.

---

# Using Structs to Structure Related Data

## Defining and Instantiating Structs

To define a struct, we enter the keyword `struct` and name the entire struct. A struct’s name should describe the
significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of
the pieces of data, which we call _fields_.

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

To get a specific value from a struct, we use dot notation. If the instance is mutable, we can change a value by using
the dot notation and assigning into a particular field. Note that the entire instance must be mutable; Rust doesn’t
allow us to mark only certain fields as mutable.

```rust
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

### Using the Field Init Shorthand

Because the parameter names and the struct field names are exactly the same, we can use the _field init shorthand_
syntax to rewrite `build_user` so it behaves exactly the same but doesn’t have the repetition of `username` and `email`.

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
```

### Creating Instances from Other Instances with Struct Update Syntax

It’s often useful to create a new instance of a struct that includes most of the values from another instance, but
changes some. You can do this using struct _update syntax_.

```rust
fn main() {
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

The syntax `..` specifies that the remaining fields not explicitly set should have the same value as the fields in the
given instance. The `..user1` must come last to specify that any remaining fields should get their values from the
corresponding fields in `user1`.

> **IMPORTANT!**
>
> In this example, we can no longer use `user1` as a whole after creating `user2` because the `String` in the `username`
> field of `user1` was moved into `user2`. If we had given `user2` new `String` values for both `email` and `username`,
> and thus only used the `active` and `sign_in_count` values from `user1`, then `user1` would still be valid after
> creating `user2`. Both `active` and `sign_in_count` are types that implement the `Copy` trait.

### Using Tuple Structs Without Named Fields to Create Different Types

Rust also supports structs that look similar to tuples, called _tuple structs_. Tuple structs have the added meaning the
struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields.

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

### Unit-Like Structs Without Any Fields

You can also define structs that don’t have any fields! These are called unit-like structs because they behave similarly
to `()`. Unit-like structs can be useful when you need to implement a trait on some type but don’t have any data that
you want to store in the type itself.

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

### Ownership of Struct Data

It’s also possible for structs to store references to data owned by something else, but to do so requires the use of
_lifetimes_. Lifetimes ensure that the data referenced by a struct is valid for as long as the struct is. Let’s say you
try to store a reference in a struct without specifying lifetimes, like the following; this won’t work:

```rust
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: "someusername123",
        email: "someone@example.com",
        sign_in_count: 1,
    };
}
```

## Method Syntax

_Methods_ are similar to functions: we declare them with the `fn` keyword and a name, they can have parameters and a
return value. Unlike functions, methods are defined within the context of a struct (or an enum or a trait object), and
their first parameter is always self, which represents the instance of the struct the method is being called on.

### Defining Methods

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

We need to use the `&` in front of the `self` shorthand to indicate that this method borrows the `Self` instance, just
as we did in rectangle: `&Rectangle`. Methods can take ownership of `self`, borrow `self` immutably, as we’ve done here,
or borrow `self` mutably, just as they can any other parameter.

### Associated Functions

All functions defined within an `impl` block are called associated functions because they’re associated with the type
named after the `impl`. We can define associated functions that don’t have `self` as their first parameter (and thus are
not methods) because they don’t need an instance of the type to work with. We’ve already used one function like this:
the `String::from` function that’s defined on the `String` type.

```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

---

# Enums and Pattern Matching

## Defining an Enum

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

We can create instances of each of the two variants of `IpAddrKind` like this:

```rust
fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
}
```

We can put data directly into each enum variant. Each variant can have different types and amounts of associated data.
Just as we’re able to define methods on structs using `impl`, we’re also able to define methods on enums.

```rust
struct Point {
    x: i32,
    y: i32,
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    MovePoint(Point),
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

fn main() {
    let quit = Message::Quit;
    let move_to = Message::Move { x: 1, y: 2 };
    let move_to_point = Message::MovePoint(Point { x: 1, y: 2 });
    let write = Message::Write("Hello!".to_string());
    let change_color = Message::ChangeColor(1, 2, 3);
}
```

### The `Option` Enum and Its Advantages Over Null Values

Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This
enum is `Option<T>`.

```rust
enum Option<T> {
    None,
    Some(T),
}

fn main() {
    let some_number = Some(5);
    let some_char = Some('e');
    let absent_number: Option<i32> = None;
}
```

## The `match` Control Flow Construct

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is
how we can extract values out of enum variants. The arms’ patterns must cover all possibilities.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

fn main() {
    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

Using enums, we can also take special actions for a few particular values, but for all other values take one default
action. This code compiles, even though we haven’t listed all the possible values a `u8` can have, because the last
pattern will match all values not specifically listed.

```rust
fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn move_player(num_spaces: u8) {}

fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }
}
```

Rust also has a pattern we can use when we want a catch-all but don’t want to use the value in the catch-all
pattern: `_` is a special pattern that matches any value and does not bind to that value. This tells Rust we aren’t
going to use the value, so Rust won’t warn us about an unused variable.

```rust
fn add_fancy_hat() {}
fn remove_fancy_hat() {}
fn reroll() {}

fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }
}
```

## Concise Control Flow with if let

The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one pattern
while ignoring the rest.

Consider the program that matches on an `Option<u8>` value in the `config_max` variable but only wants to execute code
if the value is the `Some` variant.

```rust
fn main() {
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {max}"),
        _ => (),
    }
}
```

Instead, we could write this in a shorter way using `if let`.

```rust
fn main() {
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {max}");
    }
}
```

---

# Managing Growing Projects with Packages, Crates, and Modules

Rust has a number of features that allow you to manage your code’s organization, including which details are exposed,
which details are private, and what names are in each scope in your programs. These features, sometimes collectively
referred to as the _module system_, include:

- **Packages:** A Cargo feature that lets you build, test, and share crates
- **Crates:** A tree of modules that produces a library or executable
- **Modules** and **use:** Let you control the organization, scope, and privacy of paths
- **Paths:** A way of naming an item, such as a struct, function, or module

A _crate_ is the smallest amount of code that the Rust compiler considers at a time. Even if you run `rustc` rather
than `cargo` and pass a single source code file, the compiler considers that file to be a crate. Crates can contain
modules, and the modules may be defined in other files that get compiled with the crate.

A crate can come in one of two forms: a **binary** crate or a **library** crate.

## Defining Modules to Control Scope and Privacy

### Modules Cheat Sheet

- **Start from the crate root:** When compiling a crate, the compiler first looks in the crate root file (
  usually `src/lib.rs` for a library crate or `src/main.rs` for a binary crate) for code to compile.
- **Declaring modules:** In the crate root file, you can declare new modules; say you declare a “garden” module
  with `mod garden;`. The compiler will look for the module’s code in these places:
    - Inline, within curly brackets that replace the semicolon following `mod garden`
    - In the file `src/garden.rs`
    - In the file `src/garden/mod.rs`
- **Declaring submodules:** In any file other than the crate root, you can declare submodules. For example, you might
  declare mod `vegetables`; in `src/garden.rs`. The compiler will look for the submodule’s code within the directory
  named for the parent module in these places:
    - Inline, directly following `mod vegetables`, within curly brackets instead of the semicolon
    - In the file `src/garden/vegetables.rs`
    - In the file `src/garden/vegetables/mod.rs`
- **Paths to code in modules:** Once a module is part of your crate, you can refer to code in that module from anywhere
  else in that same crate, as long as the privacy rules allow, using the path to the code. For example, an `Asparagus`
  type in the garden vegetables module would be found at `crate::garden::vegetables::Asparagus`.
- **Private vs. public:** Code within a module is private from its parent modules by default. To make a module public,
  declare it with `pub mod` instead of `mod`. To make items within a public module public as well, use `pub` before
  their declarations.
- **The `use` keyword:** Within a scope, the `use` keyword creates shortcuts to items to reduce repetition of long
  paths. In any scope that can refer to `crate::garden::vegetables::Asparagus`, you can create a shortcut
  with `use crate::garden::vegetables::Asparagus;` and from then on you only need to write `Asparagus` to make use of
  that type in the scope.

---

# Common Collections

## Storing Lists of Values with Vectors

```rust
fn main() {
    let v: Vec<i32> = Vec::new();
    let v = vec![1, 2, 3];
}
```

### Updating a Vector

```rust
fn main() {
    let mut v = Vec::new();
    v.push(5);
    v.push(4);
}
```

### Reading Elements of Vectors

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
}
```

### Iterating Over the Values in a Vector

```rust
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }

    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}
```

### Using an Enum to Store Multiple Types

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

fn main() {
    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}
```

### Dropping a Vector Drops Its Elements

Like any other struct, a vector is freed when it goes out of scope. When the vector gets dropped, all of its contents
are also dropped, meaning the integers it holds will be cleaned up. The borrow checker ensures that any references to
contents of a vector are only used while the vector itself is valid.

```rust
fn main() {
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
}
```

## Storing UTF-8 Encoded Text with Strings

```rust
fn main() {
    let s1 = String::new();
    let s2 = "initial contents".to_string();
    let s3 = String::from("initial contents");
}
```

### Updating a String

We can grow a `String` by using the `push_str` method to append a string slice. After these two lines, `s` will
contain `foobar`. The `push_str` method takes a string slice because we don’t necessarily want to take ownership of the
parameter.

```rust
fn main() {
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
}
```

The `push` method takes a single character as a parameter and adds it to the `String`.

```rust
fn main() {
    let mut s = String::from("lo");
    s.push('l');
}
```

### Concatenation with the + Operator or the format! Macro

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
}
```

The string `s3` will contain `Hello, world!`. The reason `s1` is no longer valid after the addition, and the reason we
used a reference to `s2`, has to do with the signature of the method that’s called when we use the `+` operator. The `+`
operator uses the `add` method, whose signature looks something like this:

```rust
impl String {
    fn add(self, s: &str) -> String {}
}
```

For combining strings in more complicated ways, we can instead use the `format!` macro:

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    let s = format!("{s1}-{s2}-{s3}");
}
```

### Methods for Iterating Over Strings

The best way to operate on pieces of strings is to be explicit about whether you want characters or bytes.

```rust
fn main() {
    for c in "Зд".chars() {
        println!("{c}"); // prints the characters: З, д
    }

    for b in "Зд".bytes() {
        println!("{b}"); // prints the raw bytes: 208, 151, 208, 180
    }
}
```

## Storing Keys with Associated Values in Hash Maps

By default, `HashMap` uses a hashing function called _SipHash_ that can provide resistance to denial-of-service (DoS)
attacks involving hash tables. This is not the fastest hashing algorithm available, but the trade-off for better
security that comes with the drop in performance is worth it. If you profile your code and find that the default hash
function is too slow for your purposes, you can switch to another function by specifying a different hasher.

### Creating a New Hash Map

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```

### Accessing Values in a Hash Map

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name) // `get` method returns an `Option<&V>`
        .copied() // calling `copied` to get an `Option<i32>` rather than an `Option<&i32>`
        .unwrap_or(0); // `unwrap_or` to set score to zero if scores doesn’t have an entry for the key
}    
```

We can iterate over each key–value pair in a hash map in a similar manner as we do with vectors, using a `for` loop
(this code will print each pair in an arbitrary order):

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
}
```

### Hash Maps and Ownership

For types that implement the `Copy` trait, like `i32`, the values are copied into the hash map. For owned values
like `String`, the values will be moved and the hash map will be the owner of those values.

```rust
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
}
```

### Updating a Hash Map

#### Overwriting a Value

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{scores:?}"); // {"Blue": 25}
}
```

#### Adding a Key and Value Only If a Key Isn’t Present

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{scores:?}"); // {"Yellow": 50, "Blue": 10}
}
```

#### Updating a Value Based on the Old Value

```rust
use std::collections::HashMap;

fn main() {
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0); // `or_insert` method returns a mutable reference (`&mut V`) to the value for the specified key
        *count += 1;
    }

    println!("{map:?}");
}
```

---

# Error Handling

Rust groups errors into two major categories: _recoverable_ and _unrecoverable_ errors.

Rust doesn’t have exceptions. Instead, it has the type `Result<T, E>` for recoverable errors and the `panic!` macro that
stops execution when the program encounters an unrecoverable error.

## Unrecoverable Errors with panic!

> #### Unwinding the Stack or Aborting in Response to a Panic
> By default, when a panic occurs the program starts _unwinding_, which means Rust walks back up the stack and cleans up
> the data from each function it encounters. However, walking back and cleaning up is a lot of work. Rust, therefore,
> allows you to choose the alternative of immediately _aborting_, which ends the program without cleaning up. Memory
> that
> the program was using will then need to be cleaned up by the operating system. If in your project you need to make the
> resultant binary as small as possible, you can switch from unwinding to aborting upon a panic by
> adding `panic = 'abort'` to the appropriate `[profile]` sections in your Cargo.toml file. For example, if you want to
> abort on panic in release mode, add this:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

## Recoverable Errors with Result

```rust
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {error:?}"),
    };
}
```

### Shortcuts for Panic on Error: unwrap and expect

Using `match` works well enough, but it can be a bit verbose and doesn’t always communicate intent well. The `unwrap`
method is a shortcut method implemented just like the `match` expression in the code snippet above.

```rust
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap(); // just panics on `Err`
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project"); // panics on `Err` with a custom message
}
```

### Propagating Errors

When a function’s implementation calls something that might fail, instead of handling the error within the function
itself you can return the error to the calling code so that it can decide what to do. This is known as _propagating_ the
error and gives more control to the calling code, where there might be more information or logic that dictates how the
error should be handled than what you have available in the context of your code.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

The `?` placed after a `Result` value is defined to work in almost the same way as the `match` expressions we defined to
handle the `Result` values above. If the value of the `Result` is an `Ok`, the value inside the `Ok` will get returned
from this expression, and the program will continue. If the value is an `Err`, the `Err` will be returned from the whole
function as if we had used the `return` keyword so the error value gets propagated to the calling code.

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

> As with using `?` on `Result`, you can only use `?` on `Option` in a function that returns an `Option`. The behavior
> of the `?` operator when called on an `Option<T>` is similar to its behavior when called on a `Result<T, E>`: if the
> value is `None`, the `None` will be returned early from the function at that point. If the value is `Some`, the value
> inside the `Some` is the resultant value of the expression, and the function continues.

---

# Generic Types, Traits, and Lifetimes

## Traits: Defining Shared Behavior

A _trait_ defines the functionality a particular type has and can share with other types. We can use traits to define
shared behavior in an abstract way. We can use _trait bounds_ to specify that a generic type can be any type that has
certain behavior.

```rust
pub trait Summary {
    fn summarize(&self) -> String;

    fn summarize_default_implementation(&self) -> String {
        String::from("(Read more...)")
    }
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

### Traits as Parameters

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

The `impl Trait` syntax works for straightforward cases but is actually syntax sugar for a longer form known as a trait
bound; it looks like this:

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

The `impl Trait` syntax is convenient and makes for more concise code in simple cases, while the fuller trait bound
syntax can express more complexity in other cases.

Using impl Trait is appropriate if we want this function to allow item1 and item2 to have different types (as long as
both types implement Summary).

```rust 
pub fn notify(item1: &impl Summary, item2: &impl Summary) {}
```

The generic type `T` specified as the type of the `item1` and `item2` parameters constrains the function such that the
concrete type of the value passed as an argument for `item1` and `item2` must be the same.

```rust
pub fn notify<T: Summary>(item1: &T, item2: &T) {}
```

We can also specify more than one trait bound.

```rust
pub fn notify(item: &(impl Summary + Display)) {}

pub fn notify<T: Summary + Display>(item: &T) {}
```

### Clearer Trait Bounds with where Clauses\

```rust
fn some_function<T, U>(t: &T, u: &U) -> ()
where
    T: Display + Clone,
    U: Clone + Debug,
{}
```

### Returning Types that Implement Traits

```rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```

However, you can only use impl Trait if you’re returning a single type.

## Validating References with Lifetimes

Lifetimes are another kind of generic that we’ve already been using. Rather than ensuring that a type has the behavior
we want, lifetimes ensure that references are valid as long as we need them to be.

### Preventing Dangling References with Lifetimes

The main aim of lifetimes is to prevent _dangling references_, which cause a program to reference data other than the
data it’s intended to reference.

```rust
fn main() {
    let r;                // ---------+-- 'a
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
    println!("r: {r}");   //          |
}                         // ---------+
```

### Lifetime Annotation Syntax

Lifetime annotations don’t change how long references live. Rather, they describe the relationships of the lifetimes of
multiple references to each other without affecting the lifetimes.

```
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

### Lifetime Annotations in Function Signatures

We want the signature to express the following constraint: the returned reference will be valid as long as both the
parameters are valid. This is the relationship between lifetimes of the parameters and the return value.

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

The way in which you need to specify lifetime parameters depends on what your function is doing. For example, if we
changed the implementation of the `longest` function to always return the first parameter rather than the longest string
slice, we wouldn’t need to specify a lifetime on the y parameter. The following code will compile:

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime
parameter for one of the parameters.

```rust
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str() // error: `result` does not live long enough
} // `result` dropped here while still borrowed
```

Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions.

### Lifetime Annotations in Struct Definitions

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

As with generic data types, we declare the name of the generic lifetime parameter inside angle brackets after the name
of the struct so we can use the lifetime parameter in the body of the struct definition. This annotation means an
instance of `ImportantExcerpt` can’t outlive the reference it holds in its `part` field.

### The Static Lifetime

One special lifetime we need to discuss is `'static`, which denotes that the affected reference can live for the entire
duration of the program. All string literals have the `'static` lifetime, which we can annotate as follows:

```rust
fn main() {
    let s: &'static str = "I have a static lifetime.";
}
```

---

# Writing Automated Tests
