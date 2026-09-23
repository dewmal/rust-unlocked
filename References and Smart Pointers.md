# Rust References and Smart Pointers

Rust manages memory without a garbage collector. It uses ownership, borrowing, and smart pointers to determine how long values should remain in memory.

This tutorial introduces four important tools:

- `&T` and `&mut T` for borrowing
- `Box<T>` for single ownership on the heap
- `Rc<T>` for shared ownership in one thread
- `Arc<T>` for shared ownership across threads

## 1. Ownership

Every Rust value normally has one owner. When the owner leaves its scope, Rust drops the value and releases its memory.

```rust
fn main() {
    {
        let message = String::from("Hello");
        println!("{message}");
    } // `message` is dropped here
}
```

Assigning an owned value to another variable usually moves its ownership:

```rust
fn main() {
    let first = String::from("Hello");
    let second = first;

    println!("{second}");
    // println!("{first}"); // Error: `first` no longer owns the String
}
```

This rule prevents multiple variables from accidentally freeing the same memory.

## 2. References: `&T` and `&mut T`

A reference lets code temporarily access a value without taking ownership of it. This is called borrowing.

### Immutable reference: `&T`

Use `&T` when code only needs to read a value.

```rust
fn print_length(text: &String) {
    println!("Length: {}", text.len());
}

fn main() {
    let message = String::from("Hello");

    print_length(&message);
    println!("{message}"); // The original owner can still use it
}
```

`print_length` borrows `message`. It does not own or drop it.

Rust allows multiple immutable references to the same value because none of them can modify it.

### Mutable reference: `&mut T`

Use `&mut T` when borrowed code must modify a value.

```rust
fn add_world(text: &mut String) {
    text.push_str(" world");
}

fn main() {
    let mut message = String::from("Hello");

    add_world(&mut message);
    println!("{message}");
}
```

Rust permits only one active mutable reference to a value at a time. This helps prevent data races and unexpected changes.

### When to use a reference

Use a reference when another part of the program needs temporary access, but does not need to own or keep the value alive independently.

## 3. `Box<T>`: One Owner on the Heap

`Box<T>` stores a value on the heap and gives it one owner.

```rust
fn main() {
    let number = Box::new(10);
    println!("{number}");
}
```

Conceptually, the memory looks like this:

```text
Stack                         Heap
+-----------------+           +------+
| pointer/address | --------> |  10  |
+-----------------+           +------+
      Box<i32>                    i32
```

The `Box` contains a fixed-size pointer. The actual value is stored at the heap address referenced by that pointer.

When the `Box` is dropped, Rust also drops the heap value automatically.

### Why recursive types need `Box`

Consider a linked list whose node directly contains another node:

```rust,ignore
enum List {
    Node(i32, List),
    Empty,
}
```

Rust cannot calculate the size of `List`:

```text
List size = i32 + List size
          = i32 + i32 + List size
          = ...
```

The calculation never ends. Using `Box` replaces the nested value with a fixed-size pointer:

```rust
enum List {
    Node(i32, Box<List>),
    Empty,
}

fn main() {
    let list = List::Node(
        10,
        Box::new(List::Node(
            20,
            Box::new(List::Empty),
        )),
    );
}
```

Each node now contains an integer and a fixed-size pointer to the next node.

### Other uses for `Box`

`Box<T>` is also useful for:

- Moving a large value to the heap
- Trait objects such as `Box<dyn Animal>`
- Transferring ownership without copying the underlying value

Use `Box<T>` when the value needs exactly one owner and heap allocation is useful or required.

## 4. `Rc<T>`: Shared Ownership in One Thread

`Rc` means **Reference Counted**. It lets multiple variables own the same value in a single-threaded program.

```rust
use std::rc::Rc;

fn main() {
    let first = Rc::new(String::from("Hello"));
    println!("Count: {}", Rc::strong_count(&first)); // 1

    {
        let second = Rc::clone(&first);
        println!("Count: {}", Rc::strong_count(&first)); // 2
        println!("{second}");
    }

    println!("Count: {}", Rc::strong_count(&first)); // 1
}
```

The reference count changes as follows:

1. `Rc::new` creates the first owner, so the count is `1`.
2. `Rc::clone` creates another owner, so the count becomes `2`.
3. When `second` is dropped, the count returns to `1`.
4. When the last owner is dropped, the count becomes `0` and the value is freed.

`Rc::clone` does not copy the `String`. It only creates another owner and increases the count.

### When to use `Rc`

Use `Rc<T>` when:

- Several parts of one thread must own the same value
- No single owner naturally controls the value's lifetime
- Borrowing is insufficient because each owner must keep the value alive

Common examples include graphs, shared tree nodes, and shared application data.

Do not send an `Rc<T>` between threads. Its reference count is not thread-safe.

## 5. `Arc<T>`: Shared Ownership Across Threads

`Arc` means **Atomically Reference Counted**. It provides shared ownership like `Rc`, but its counter can be updated safely by multiple threads.

```rust
use std::sync::Arc;
use std::thread;

fn main() {
    let message = Arc::new(String::from("Hello from Arc"));
    let worker_message = Arc::clone(&message);

    let handle = thread::spawn(move || {
        println!("Worker: {worker_message}");
    });

    println!("Main: {message}");
    handle.join().unwrap();
}
```

Both the main thread and worker thread own the same `String`. The value remains alive until both `Arc` owners are dropped.

Atomic reference counting has a small runtime cost. Therefore, prefer `Rc<T>` when sharing is limited to one thread.

## 6. Shared Mutation with `Arc<Mutex<T>>`

`Arc<T>` provides shared ownership, but it does not automatically allow several threads to modify the value safely.

A `Mutex<T>` allows only one thread at a time to access the protected value:

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = Vec::new();

    for _ in 0..5 {
        let shared_counter = Arc::clone(&counter);

        let handle = thread::spawn(move || {
            let mut value = shared_counter.lock().unwrap();
            *value += 1;
        });

        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Counter: {}", *counter.lock().unwrap()); // 5
}
```

In this combination:

- `Arc` lets the threads share ownership of the counter.
- `Mutex` controls mutable access to the counter.

## 7. Reference Cycles

Reference counting has an important limitation. If two `Rc` values strongly reference each other, neither count may reach zero.

```text
Object A --strong reference--> Object B
Object A <--strong reference-- Object B
```

This creates a reference cycle and can cause a memory leak.

Rust provides `Weak<T>` for relationships that should not own the referenced value. A weak reference does not increase the strong count and therefore does not keep the value alive.

Use strong `Rc` or `Arc` references for ownership and `Weak` references to break cycles, such as parent pointers in a tree.

## 8. Quick Comparison

| Type | Ownership | Thread support | Typical use |
|---|---|---|---|
| `&T` | No ownership; immutable borrow | Depends on the borrowed type | Temporary read access |
| `&mut T` | No ownership; mutable borrow | Depends on the borrowed type | Temporary write access |
| `Box<T>` | One owner | Can move across threads when `T` permits it | Heap allocation and recursive types |
| `Rc<T>` | Multiple owners | Single thread only | Shared ownership in one thread |
| `Arc<T>` | Multiple owners | Multiple threads | Thread-safe shared ownership |

## 9. How to Choose

Ask these questions in order:

1. Does the code only need temporary access? Use `&T` or `&mut T`.
2. Does the heap value need only one owner? Use `Box<T>`.
3. Does it need multiple owners in one thread? Use `Rc<T>`.
4. Does it need multiple owners across threads? Use `Arc<T>`.
5. Must several threads modify it? Consider `Arc<Mutex<T>>` or another synchronization type.

Prefer normal ownership and borrowing when possible. Use smart pointers when the data structure or lifetime requirements need them.

## Summary

```text
&T / &mut T   -> Temporary borrowed access
Box<T>        -> One owner; value stored on the heap
Rc<T>         -> Multiple owners within one thread
Arc<T>        -> Multiple owners across threads
Arc<Mutex<T>> -> Shared ownership and synchronized mutation
```

The key difference is not simply where the data is stored. The important question is **who owns the value and how long it must remain alive**.
