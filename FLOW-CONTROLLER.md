# Rust Basics: Variables, Operators, and Flow Control

This guide introduces Rust variables and operators before explaining the keywords used to control program flow.

## 1. Variables

A variable stores a value. Use the `let` keyword to create one:

```rust
fn main() {
    let age = 20;
    let name = "Alex";

    println!("Name: {name}");
    println!("Age: {age}");
}
```

Rust usually determines the variable’s type automatically:

```rust
let age = 20;       // i32 integer
let price = 9.99;   // f64 floating-point number
let active = true;  // bool
let letter = 'R';   // char
let name = "Rust";  // &str
```

### Type annotations

You can explicitly state a variable’s type using `:`:

```rust
let age: i32 = 20;
let price: f64 = 9.99;
let active: bool = true;
```

### Immutable variables

Variables are immutable by default, meaning their values cannot be changed:

```rust
let number = 10;

// number = 20; // Error
```

### Mutable variables

Add the `mut` keyword when a variable’s value needs to change:

```rust
fn main() {
    let mut number = 10;

    number = 20;

    println!("{number}");
}
```

### Constants

Use `const` for a value that must never change:

```rust
const MAX_USERS: u32 = 100;
```

Constants require an explicit type and are normally written using uppercase letters.

### Shadowing

Rust allows a new variable to reuse an existing variable’s name:

```rust
fn main() {
    let number = 5;
    let number = number + 1;
    let number = number * 2;

    println!("{number}"); // 12
}
```

This is called shadowing. It creates a new variable rather than changing the original one.

---

## 2. Arithmetic operators

Arithmetic operators perform mathematical calculations.

```rust
fn main() {
    let a = 10;
    let b = 3;

    println!("{}", a + b); // Addition: 13
    println!("{}", a - b); // Subtraction: 7
    println!("{}", a * b); // Multiplication: 30
    println!("{}", a / b); // Integer division: 3
    println!("{}", a % b); // Remainder: 1
}
```

| Operator | Meaning | Example |
|---|---|---|
| `+` | Addition | `a + b` |
| `-` | Subtraction | `a - b` |
| `*` | Multiplication | `a * b` |
| `/` | Division | `a / b` |
| `%` | Remainder | `a % b` |

When both operands are integers, `/` performs integer division:

```rust
let result = 10 / 3; // 3
```

Use floating-point numbers when you need a decimal result:

```rust
let result = 10.0 / 3.0; // approximately 3.333
```

---

## 3. Comparison operators

Comparison operators compare two values and produce a `bool` value: either `true` or `false`.

```rust
fn main() {
    let a = 10;
    let b = 5;

    println!("{}", a == b); // false
    println!("{}", a != b); // true
    println!("{}", a > b);  // true
    println!("{}", a < b);  // false
    println!("{}", a >= b); // true
    println!("{}", a <= b); // false
}
```

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |

Do not confuse assignment with equality:

```rust
let number = 10;       // = assigns a value
let result = number == 10; // == compares values
```

---

## 4. Logical operators

Logical operators combine or reverse Boolean conditions.

```rust
fn main() {
    let age = 20;
    let has_id = true;

    let can_enter = age >= 18 && has_id;
    let needs_help = age < 18 || !has_id;

    println!("Can enter: {can_enter}");
    println!("Needs help: {needs_help}");
}
```

| Operator | Meaning |
|---|---|
| `&&` | AND: both conditions must be true |
| `||` | OR: at least one condition must be true |
| `!` | NOT: reverses a Boolean value |

Example:

```rust
let result = true && false; // false
let result = true || false; // true
let result = !true;         // false
```

---

## 5. Assignment operators

Assignment operators update mutable variables:

```rust
fn main() {
    let mut number = 10;

    number += 5; // number = number + 5
    number -= 2; // number = number - 2
    number *= 3; // number = number * 3
    number /= 2; // number = number / 2
    number %= 4; // number = number % 4

    println!("{number}");
}
```

| Operator | Meaning |
|---|---|
| `=` | Assign a value |
| `+=` | Add and assign |
| `-=` | Subtract and assign |
| `*=` | Multiply and assign |
| `/=` | Divide and assign |
| `%=` | Calculate remainder and assign |

---

# Flow Control in Rust

Flow control determines which code runs, how often it runs, and when a function or loop stops.

## 6. `if`

The `if` keyword runs code only when its condition is `true`:

```rust
fn main() {
    let age = 20;

    if age >= 18 {
        println!("You are an adult.");
    }
}
```

An `if` condition must produce a `bool`.

---

## 7. `else`

The `else` keyword provides an alternative when an `if` condition is `false`:

```rust
fn main() {
    let number = 7;

    if number % 2 == 0 {
        println!("The number is even.");
    } else {
        println!("The number is odd.");
    }
}
```

---

## 8. `else if`

Use `else if` to test several conditions in order:

```rust
fn main() {
    let score = 75;

    if score >= 90 {
        println!("Grade A");
    } else if score >= 75 {
        println!("Grade B");
    } else if score >= 50 {
        println!("Grade C");
    } else {
        println!("Try again");
    }
}
```

Rust stops after finding the first condition that is `true`.

### Using `if` as an expression

An `if` expression can produce a value:

```rust
fn main() {
    let age = 20;

    let status = if age >= 18 {
        "adult"
    } else {
        "minor"
    };

    println!("Status: {status}");
}
```

Both branches must produce compatible types.

---

## 9. `match`

The `match` keyword compares a value against several patterns:

```rust
fn main() {
    let day = 2;

    match day {
        1 => println!("Monday"),
        2 => println!("Tuesday"),
        3 => println!("Wednesday"),
        _ => println!("Another day"),
    }
}
```

Important syntax:

- `match` starts the comparison.
- `=>` separates a pattern from its code.
- `,` separates each match arm.
- `_` matches any remaining value.

A `match` must cover every possible case.

The `|` symbol means “or” inside a pattern:

```rust
fn main() {
    let number = 2;

    match number {
        1 | 2 | 3 => println!("The number is between 1 and 3."),
        _ => println!("The number is outside that range."),
    }
}
```

---

## 10. `loop`

The `loop` keyword repeats code indefinitely:

```rust
fn main() {
    let mut count = 1;

    loop {
        println!("{count}");
        count += 1;

        if count > 5 {
            break;
        }
    }
}
```

Without `break`, this loop would run forever.

A `loop` can also produce a value:

```rust
fn main() {
    let mut number = 0;

    let result = loop {
        number += 1;

        if number == 5 {
            break number * 2;
        }
    };

    println!("Result: {result}");
}
```

Here, `break number * 2` ends the loop and returns `10`.

---

## 11. `while`

The `while` keyword repeats code while a condition remains `true`:

```rust
fn main() {
    let mut number = 1;

    while number <= 5 {
        println!("{number}");
        number += 1;
    }
}
```

The condition is checked before every iteration.

---

## 12. `for`

The `for` keyword loops over values from an iterator, such as a range or collection:

```rust
fn main() {
    for number in 1..=5 {
        println!("{number}");
    }
}
```

Important parts:

- `for` begins the loop.
- `number` stores the current value.
- `in` connects the variable to the iterator.
- `1..=5` includes the numbers `1` through `5`.
- `1..5` includes the numbers `1` through `4`.

You can also loop over a collection:

```rust
fn main() {
    let names = ["Amy", "Ben", "Cara"];

    for name in names {
        println!("Hello, {name}!");
    }
}
```

---

## 13. `break`

The `break` keyword immediately exits the nearest loop:

```rust
fn main() {
    for number in 1..=10 {
        if number == 4 {
            break;
        }

        println!("{number}");
    }
}
```

Output:

```text
1
2
3
```

---

## 14. `continue`

The `continue` keyword skips the rest of the current iteration:

```rust
fn main() {
    for number in 1..=5 {
        if number == 3 {
            continue;
        }

        println!("{number}");
    }
}
```

Output:

```text
1
2
4
5
```

---

## 15. `return`

The `return` keyword immediately exits the current function:

```rust
fn check_number(number: i32) {
    if number < 0 {
        println!("Negative numbers are not allowed.");
        return;
    }

    println!("The number is {number}.");
}

fn main() {
    check_number(-5);
}
```

Rust can also return the final expression automatically when it does not end with a semicolon:

```rust
fn double(number: i32) -> i32 {
    number * 2
}
```

---

## 16. Loop labels

A loop label lets you control an outer loop from inside a nested loop. Labels begin with an apostrophe:

```rust
fn main() {
    'outer: for row in 1..=3 {
        for column in 1..=3 {
            if row == 2 && column == 2 {
                break 'outer;
            }

            println!("Row: {row}, Column: {column}");
        }
    }
}
```

`break 'outer` exits the loop marked with `'outer`.

---

# Keyword summary

| Keyword | Purpose |
|---|---|
| `let` | Creates a variable |
| `mut` | Allows a variable’s value to change |
| `const` | Creates a constant |
| `if` | Runs code when a condition is true |
| `else` | Runs alternative code |
| `match` | Selects code using patterns |
| `loop` | Repeats indefinitely |
| `while` | Repeats while a condition is true |
| `for` | Iterates over a range or collection |
| `in` | Connects a loop variable to an iterator |
| `break` | Exits a loop |
| `continue` | Skips to the next loop iteration |
| `return` | Exits a function |
| `'label` | Identifies a loop in nested loops |

# Star-mark printing example

The following program prints a right-angled triangle:

```rust
fn main() {
    let rows = 5;

    for row in 1..=rows {
        for _ in 1..=row {
            print!("* ");
        }

        println!();
    }
}
```

Output:

```text
*
* *
* * *
* * * *
* * * * *
```

How it works:

1. `let rows = 5` stores the number of rows.
2. The outer `for` loop controls the current row.
3. The inner `for` loop controls the number of stars.
4. `_` means the current inner-loop value is intentionally unused.
5. `print!("* ")` prints a star without starting a new line.
6. `println!()` starts a new line after each row.
7. The range `1..=row` makes each row print one more star than the previous row.
