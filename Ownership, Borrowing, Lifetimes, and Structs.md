# Rust Ownership, Borrowing, Lifetimes, and Structs

## Introduction

Rust is designed to provide memory safety without requiring a garbage collector. It achieves this primarily through three connected concepts:

1. **Ownership** - determines who is responsible for a value.
2. **Borrowing** - allows code to use a value without taking ownership.
3. **Lifetimes** - ensure that borrowed references never outlive the values they refer to.

Rust checks these rules at compile time. If a program violates them, it normally fails to compile instead of producing a memory error while running.

These concepts help prevent problems such as:

- Using memory after it has been freed.
- Freeing the same memory more than once.
- Keeping dangling references.
- Modifying the same data unsafely from multiple places.
- Data races in concurrent programs.

---

## 1. Ownership

Every value in Rust has an **owner**. The owner is usually the variable that currently holds the value.

```rust
fn main() {
    let name = String::from("Kamal");
    println!("{name}");
}
```

Here, `name` owns the `String` value.

When `name` goes out of scope at the end of `main`, Rust automatically drops the `String` and releases its memory.

### The three ownership rules

Rust ownership can be summarized using three rules:

1. Every value has an owner.
2. A value has only one owner at a time.
3. When the owner goes out of scope, the value is dropped.

### Scope and automatic cleanup

```rust
fn main() {
    {
        let message = String::from("Hello");
        println!("{message}");
    } // message is dropped here
}
```

The `message` variable is valid only inside the inner block. Rust releases its memory when that block ends.

This is sometimes called **RAII**: Resource Acquisition Is Initialization. A resource is connected to the lifetime of its owner and is cleaned up when the owner is dropped.

---

## 2. Stack and Heap Data

Ownership becomes easier to understand when we distinguish stack data from heap data.

### Stack data

Values with a fixed, known size can usually be stored directly on the stack.

Examples include:

- Integers such as `i32`.
- Floating-point numbers such as `f64`.
- Booleans.
- Characters.
- Tuples containing only `Copy` values.

```rust
let number: i32 = 10;
let enabled: bool = true;
```

These values are normally inexpensive to copy.

### Heap data

Values whose size may change or is not known at compile time often store their contents on the heap.

Examples include:

- `String`
- `Vec<T>`
- `Box<T>`

```rust
let name = String::from("Kamal");
```

The `String` value contains stack metadata that points to text stored on the heap. Rust uses ownership to decide who must release that heap allocation.

---

## 3. Move Semantics

When a non-`Copy` value is assigned to another variable, Rust normally **moves** its ownership.

```rust
fn main() {
    let first = String::from("Hello");
    let second = first;

    println!("{second}");
    // println!("{first}"); // compile error
}
```

The ownership of the `String` moves from `first` to `second`. After the move, `first` is no longer valid.

Rust does this to prevent both variables from trying to free the same heap allocation.

### Moving values into functions

Passing a non-`Copy` value to a function can also move ownership.

```rust
fn display(message: String) {
    println!("{message}");
}

fn main() {
    let message = String::from("Hello");
    display(message);

    // message is no longer available here
}
```

The function parameter becomes the new owner. The value is dropped when the function ends unless the function returns it or moves it elsewhere.

### Returning ownership

```rust
fn return_message(message: String) -> String {
    message
}

fn main() {
    let message = String::from("Hello");
    let message = return_message(message);

    println!("{message}");
}
```

This works, but borrowing is usually more convenient when a function only needs temporary access.

---

## 4. Copy and Clone

### The `Copy` trait

Some types implement the `Copy` trait. Assigning a `Copy` value creates a separate value instead of moving ownership.

```rust
fn main() {
    let first = 10;
    let second = first;

    println!("{first}");
    println!("{second}");
}
```

Both variables remain valid because `i32` implements `Copy`.

Common `Copy` types include:

- Integer types.
- Floating-point types.
- `bool`.
- `char`.
- Shared references such as `&T`.
- Tuples containing only `Copy` values.

Types that manage heap allocations, such as `String` and `Vec<T>`, do not normally implement `Copy`.

### Explicit copying with `clone`

Use `clone()` when a separate copy of a non-`Copy` value is genuinely needed.

```rust
fn main() {
    let first = String::from("Hello");
    let second = first.clone();

    println!("{first}");
    println!("{second}");
}
```

Cloning may copy heap data and can therefore be more expensive than a normal move.

Use `clone()` deliberately. Do not use it only to avoid understanding an ownership error.

---

## 5. Borrowing and References

Borrowing allows code to access a value without taking ownership.

A reference points to an existing value.

```rust
fn display(message: &String) {
    println!("{message}");
}

fn main() {
    let message = String::from("Hello");

    display(&message);
    println!("{message}");
}
```

`display` receives `&String`, so it borrows the value. The ownership remains with `message` in `main`.

### Prefer `&str` when only text is needed

A function that only needs to read text often accepts `&str` instead of `&String`.

```rust
fn display(message: &str) {
    println!("{message}");
}

fn main() {
    let owned = String::from("Hello");

    display(&owned);
    display("Static text");
}
```

This is more flexible because both `String` slices and string literals can be passed to the function.

---

## 6. Immutable References: `&T`

An immutable reference allows reading but not modifying the borrowed value.

```rust
fn length(text: &String) -> usize {
    text.len()
}

fn main() {
    let text = String::from("Rust");
    let size = length(&text);

    println!("{text} has {size} bytes");
}
```

Multiple immutable references can exist at the same time:

```rust
fn main() {
    let text = String::from("Rust");

    let first = &text;
    let second = &text;

    println!("{first} {second}");
}
```

This is safe because none of the references can change the value.

---

## 7. Mutable References: `&mut T`

A mutable reference allows borrowed data to be changed.

```rust
fn add_language_name(text: &mut String) {
    text.push_str(" language");
}

fn main() {
    let mut text = String::from("Rust");
    add_language_name(&mut text);

    println!("{text}");
}
```

Three parts are important:

```rust
let mut text = String::from("Rust"); // mutable owner
```

```rust
fn add_language_name(text: &mut String) // mutable-reference parameter
```

```rust
add_language_name(&mut text); // mutable borrow
```

### The borrowing rule

For the same value at the same time, Rust allows either:

- Any number of immutable references, or
- Exactly one mutable reference.

Rust does not allow active mutable and immutable references to overlap.

```rust
fn main() {
    let mut text = String::from("Rust");

    let first = &mut text;
    first.push_str(" language");

    // The first borrow is no longer used, so another can begin.
    let second = &mut text;
    second.push('!');

    println!("{text}");
}
```

Modern Rust usually ends a borrow after its final use, not necessarily at the closing brace. This behavior is called **non-lexical lifetimes**.

---

## 8. Dereferencing with `*`

The `*` operator accesses the value pointed to by a reference.

```rust
fn main() {
    let number = 10;
    let reference = &number;

    println!("{}", *reference);
}
```

With a mutable reference, dereferencing can modify the original value:

```rust
fn main() {
    let mut number = 10;
    let reference = &mut number;

    *reference = 20;

    println!("{number}");
}
```

### Multiple reference layers

References may point to other references.

```rust
fn main() {
    let number = 10;

    let first = &number; // &i32
    let second = &first; // &&i32
    let third = &second; // &&&i32

    let value = ***third;
    println!("{value}");
}
```

Each `*` removes one reference layer:

```text
&&&i32 -> &&i32 -> &i32 -> i32
```

Rust often performs automatic referencing and dereferencing in method calls, so explicit `*` is not always necessary.

---

## 9. Lifetimes

A lifetime represents the period during which a reference is valid.

The central rule is:

> The referenced value must remain valid for at least as long as the reference is used.

### A valid reference

```rust
fn main() {
    let name = String::from("Kamal");
    let reference = &name;

    println!("{reference}");
}
```

The owner, `name`, is still alive when `reference` is used.

### A dangling-reference attempt

```rust,compile_fail
fn main() {
    let reference;

    {
        let name = String::from("Kamal");
        reference = &name;
    } // name is dropped here

    println!("{reference}");
}
```

Rust rejects this program because `reference` would point to a destroyed `String`.

### Lifetime annotations

Lifetime annotations describe relationships between references. They do not make a value live longer.

```rust
fn longest<'a>(first: &'a str, second: &'a str) -> &'a str {
    if first.len() >= second.len() {
        first
    } else {
        second
    }
}
```

`'a` means the returned reference is connected to the valid overlap of the two input references.

In practice, the returned reference cannot be used after either required input lifetime has ended.

### Lifetime elision

Rust can infer many common lifetime relationships, so annotations are often unnecessary.

```rust
fn first_word(text: &str) -> &str {
    text.split_whitespace().next().unwrap_or("")
}
```

The compiler understands that the returned slice is borrowed from `text`.

---

## 10. Structs

A `struct` groups related values into a custom data type.

```rust
struct User {
    name: String,
    age: u32,
    active: bool,
}
```

Creating a struct value:

```rust
fn main() {
    let user = User {
        name: String::from("Kamal"),
        age: 25,
        active: true,
    };

    println!("{} is {} years old", user.name, user.age);
}
```

The `user` value owns its `String` field. When `user` is dropped, its owned fields are dropped as well.

### Mutating a struct

The entire struct binding must be mutable before one of its fields can be changed.

```rust
fn main() {
    let mut user = User {
        name: String::from("Kamal"),
        age: 25,
        active: true,
    };

    user.age = 26;
}
```

---

## 11. Types of Structs

### Named-field struct

```rust
struct User {
    name: String,
    age: u32,
}
```

Named fields make the meaning of each value clear.

### Tuple struct

```rust
struct Point(i32, i32);

fn main() {
    let point = Point(10, 20);
    println!("{} {}", point.0, point.1);
}
```

Tuple structs are useful when field names are unnecessary or when creating a distinct type around another value.

```rust
struct Meters(f64);
struct Kilometers(f64);
```

Although both contain `f64`, Rust treats them as different types.

### Unit struct

```rust
struct Marker;

fn main() {
    let marker = Marker;
}
```

A unit struct stores no fields. It can be useful as a marker type or as a type that implements behavior without storing data.

---

## 12. Struct Methods and `self`

Methods are defined inside an `impl` block.

```rust
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
    let rectangle = Rectangle {
        width: 10,
        height: 5,
    };

    println!("Area: {}", rectangle.area());
}
```

### `&self`

```rust
fn area(&self) -> u32
```

The method immutably borrows the struct. It can read fields but cannot normally modify them.

### `&mut self`

```rust
impl Rectangle {
    fn double_width(&mut self) {
        self.width *= 2;
    }
}
```

The method mutably borrows the struct and can modify its fields.

### `self`

```rust
impl Rectangle {
    fn consume(self) {
        println!("Rectangle consumed");
    }
}
```

The method takes ownership of the struct. The original variable cannot normally be used after the method call.

---

## 13. Structs and Lifetimes

### A struct that owns its data

```rust
struct OwnedUser {
    name: String,
}
```

This struct owns its `String`. It needs no lifetime annotation because the data belongs to the struct.

### A struct that borrows data

```rust
struct BorrowedUser<'a> {
    name: &'a str,
}
```

This struct does not own the text. It stores a reference to text owned elsewhere.

The `'a` annotation means that the referenced text must remain valid while the struct's `name` reference is used.

```rust
fn main() {
    let name = String::from("Kamal");

    let user = BorrowedUser {
        name: &name,
    };

    println!("{}", user.name);
}
```

Rust prevents the borrowed struct from outliving the original data.

### Owned data or borrowed data?

Use owned fields when:

- The struct must keep the data independently.
- The original data may be destroyed before the struct.
- Simpler APIs are more important than avoiding a copy or allocation.

Use borrowed fields when:

- The struct is a temporary view over existing data.
- Avoiding duplication matters.
- The lifetime relationship is clear and manageable.

---

## 14. Error Handling with `Result`

Rust uses `Result<T, E>` for recoverable errors.

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

- `Ok(T)` represents success and contains a value.
- `Err(E)` represents failure and contains an error.

### Handling both cases with `match`

```rust
use std::fs::File;

fn main() {
    match File::open("data.txt") {
        Ok(file) => println!("File opened: {file:?}"),
        Err(error) => println!("Could not open file: {error}"),
    }
}
```

The function returns a `Result`, so the program must consider success and failure before accessing the file.

### Propagating an error with `?`

```rust
use std::fs::File;
use std::io;

fn open_file() -> Result<File, io::Error> {
    let file = File::open("data.txt")?;
    Ok(file)
}
```

The `?` operator:

- Extracts the value when the result is `Ok`.
- Returns the error early when the result is `Err`.

This does not ignore the error. It transfers responsibility to the caller.

### `unwrap` and `expect`

```rust
let file = File::open("data.txt").expect("data.txt must exist");
```

`unwrap()` and `expect()` panic if the result is `Err`. They can be useful in tests, prototypes, and situations where failure is truly unrecoverable.

In production code, prefer deliberate error handling or propagation when recovery is possible.

---

## 15. How the Concepts Work Together

Consider the following example:

```rust
struct Profile {
    name: String,
}

impl Profile {
    fn rename(&mut self, new_name: &str) {
        self.name.clear();
        self.name.push_str(new_name);
    }

    fn name(&self) -> &str {
        &self.name
    }
}

fn main() {
    let mut profile = Profile {
        name: String::from("Kamal"),
    };

    profile.rename("Nimal");
    println!("{}", profile.name());
}
```

This example combines several concepts:

- `profile` owns the `Profile` value.
- `Profile` owns its `String` field.
- `rename` uses `&mut self` because it changes the struct.
- `new_name` is borrowed as `&str`, so the method does not take ownership of it.
- `name` uses `&self` because it only reads the struct.
- The returned `&str` is valid only while the relevant borrow of `profile` remains valid.

---

## 16. Common Mistakes

### Mistake 1: Using a value after it has moved

```rust,compile_fail
let first = String::from("Hello");
let second = first;

println!("{first}");
```

Possible solutions:

- Use `second` as the new owner.
- Borrow with `&first` instead of moving.
- Clone only if a genuine second owned value is required.

### Mistake 2: Creating overlapping mutable borrows

```rust,compile_fail
let mut text = String::from("Rust");

let first = &mut text;
let second = &mut text;

println!("{first} {second}");
```

Use one mutable borrow at a time.

### Mistake 3: Returning a reference to a local value

```rust,compile_fail
fn invalid_name() -> &str {
    let name = String::from("Kamal");
    &name
}
```

`name` is destroyed when the function returns. Return an owned `String` instead:

```rust
fn valid_name() -> String {
    String::from("Kamal")
}
```

### Mistake 4: Adding lifetime annotations to fix unrelated errors

Lifetime annotations describe relationships. They cannot force local data to live longer and cannot repair an invalid ownership design by themselves.

### Mistake 5: Cloning everything

Excessive cloning may hide the intended ownership structure and perform unnecessary allocations. First consider borrowing or transferring ownership.

---

## 17. Quick Revision Table

| Concept | Question to remember |
|---|---|
| Ownership | Who is responsible for this value? |
| Scope | When will the owner be dropped? |
| Move | Was ownership transferred to another variable or function? |
| Copy | Was a small `Copy` value duplicated automatically? |
| Clone | Was a separate owned copy explicitly created? |
| Borrowing | Is the value being used without taking ownership? |
| `&T` | Is this a read-only borrow? |
| `&mut T` | Is this an exclusive mutable borrow? |
| `*` | Is the value behind a reference being accessed? |
| Lifetime | Does the original value remain valid while the reference is used? |
| Struct | Are related fields grouped into one custom type? |
| `&self` | Does the method only need to read the struct? |
| `&mut self` | Does the method need to modify the struct? |
| `self` | Does the method need to take ownership of the struct? |
| `Result<T, E>` | Are both success and failure being considered? |

---

## 18. Final Summary

Rust's memory model can be remembered as a sequence of questions:

1. **Who owns the value?**
2. **Was the value moved, copied, or cloned?**
3. **Can the operation borrow the value instead?**
4. **Should the borrow be immutable or mutable?**
5. **Will the original value remain alive while the reference is used?**
6. **Does a struct own its fields or borrow them?**
7. **How will an operation report and handle failure?**

The central relationship is:

```text
Ownership + Borrowing + Lifetimes = Compile-time memory safety
```

Rust uses these rules to provide safe and predictable memory management without a garbage collector.
