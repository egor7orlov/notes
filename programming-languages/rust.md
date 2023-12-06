# Rust Programming language

---

## Common Programming Concepts

### Variables and Mutability

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
