## Box

`Box` is a very convenient type in Rust. When you use a `Box`, you can put a type on the heap instead of the stack. To make a new `Box`, just use `Box::new()` and put the item inside.

```rust
fn just_takes_a_variable<T>(item: T) {} // Takes anything and drops it.

fn main() {
    let my_number = 1; // This is an i32
    just_takes_a_variable(my_number);
    just_takes_a_variable(my_number); // Using this function twice is no problem, because it's Copy

    let my_box = Box::new(1); // This is a Box<i32>
    just_takes_a_variable(my_box.clone()); // Without .clone() the second function would make an error
    just_takes_a_variable(my_box); // because Box is not Copy
}
```

At first it is hard to imagine where to use it, but you use it in Rust a lot. You remember that `&` is used for `str` because the compiler doesn't know the size of a `str`: it can be any length. But the `&` reference is always the same length, so the compiler can use it. `Box` is similar. Also, you can use `*` on a `Box` to get to the value, just like with `&`:

```rust
fn main() {
    let my_box = Box::new(1); // This is a Box<i32>
    let an_integer = *my_box; // This is an i32
    println!("{:?}", my_box);
    println!("{:?}", an_integer);
}
```

This is why Box is called a "smart pointer", because it is like a `&` reference (a kind of pointer) but can do more things.

You can also use a Box to create structs with the same struct inside. These are called *recursive*, which means that inside Struct A is maybe another Struct A. Sometimes you can use Boxes to create linked lists, although these lists are not very popular in Rust. But if you want to create a recursive struct, you can use a `Box`. Here's what happens if you try without a `Box`:


```rust
struct List {
    item: Option<List>, // ⚠️
}
```

This simple `List` has one item, that may be `Some<List>` (another list), or `None`. Because you can choose `None`, it will not be recursive forever. But the compiler still doesn't know the size:

```text
error[E0072]: recursive type `List` has infinite size
  --> src\main.rs:16:1
   |
16 | struct List {
   | ^^^^^^^^^^^ recursive type has infinite size
17 |     item: Option<List>,
   |     ------------------ recursive without indirection
   |
   = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `List` representable
```

You can see that it even suggests trying a `Box`. So let's put a `Box` around List:

```rust
struct List {
    item: Option<Box<List>>,
}
fn main() {}
```

Now the compiler is fine with the `List`, because everything is behind a `Box`, and it knows the size of a `Box`. Then a very simple list might look like this:

```rust
struct List {
    item: Option<Box<List>>,
}

impl List {
    fn new() -> List {
        List {
            item: Some(Box::new(List { item: None })),
        }
    }
}

fn main() {
    let mut my_list = List::new();
}
```

Even without data it is a bit complicated, and Rust does not use this type of pattern very much. This is because Rust has strict rules on borrowing and ownership, as you know. But if you want to start a list like this (a linked list), `Box` can help.

A `Box` also lets you use `std::mem::drop` on it, because it's on the heap. That can be convenient sometimes.

