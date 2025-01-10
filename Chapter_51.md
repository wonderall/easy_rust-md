## Reading Rust documentation

It's important to know how to read documentation in Rust so you can understand what other people wrote. Here are some things to know in Rust documentation:

### assert_eq!

You saw that `assert_eq!` is used when doing testing. You put two items inside the function and the program will panic if they are not equal. Here is a simple example where we need an even number.

```rust
fn main() {
    prints_number(56);
}

fn prints_number(input: i32) {
    assert_eq!(input % 2, 0); // number must be equal.
                              // If number % 2 is not 0, it panics
    println!("The number is not odd. It is {}", input);
}
```

Maybe you don't have any plans to use `assert_eq!` in your code, but it is everywhere in Rust documentation. This is because in a document you would need a lot of room to `println!` everything. Also, you would require `Display` or `Debug` for the things you want to print. That's why documentation has `assert_eq!` everywhere. Here is an example from here [https://doc.rust-lang.org/std/vec/struct.Vec.html](https://doc.rust-lang.org/std/vec/struct.Vec.html) showing how to use a Vec:

```rust
fn main() {
    let mut vec = Vec::new();
    vec.push(1);
    vec.push(2);

    assert_eq!(vec.len(), 2);
    assert_eq!(vec[0], 1);

    assert_eq!(vec.pop(), Some(2));
    assert_eq!(vec.len(), 1);

    vec[0] = 7;
    assert_eq!(vec[0], 7);

    vec.extend([1, 2, 3].iter().copied());

    for x in &vec {
        println!("{}", x);
    }
    assert_eq!(vec, [7, 1, 2, 3]);
}
```

In these examples, you can just think of `assert_eq!(a, b)` as saying "a is b". Now look at the same example with comments on the right. The comments show what it actually means.

```rust
fn main() {
    let mut vec = Vec::new();
    vec.push(1);
    vec.push(2);

    assert_eq!(vec.len(), 2); // "The vec length is 2"
    assert_eq!(vec[0], 1); // "vec[0] is 1"

    assert_eq!(vec.pop(), Some(2)); // "When you use .pop(), you get Some()"
    assert_eq!(vec.len(), 1); // "The vec length is now 1"

    vec[0] = 7;
    assert_eq!(vec[0], 7); // "Vec[0] is 7"

    vec.extend([1, 2, 3].iter().copied());

    for x in &vec {
        println!("{}", x);
    }
    assert_eq!(vec, [7, 1, 2, 3]); // "The vec now has [7, 1, 2, 3]"
}
```

### Searching

The top bar of a Rust document is the search bar. It shows you results as you type. When you go down a page you can't see the search bar anymore, but if you press the **s** key on the keyboard you can search again. So pressing **s** anywhere lets you search right away.

### [src] button

Usually the code for a method, struct, etc. will not be complete. This is because you don't usually need to see the full source to know how it works, and the full code can be confusing. But if you want to know more, you can click on [src] and see everything. For example, on the page for `String` you can see this signature for `.with_capacity()`:

```rust
// 🚧
pub fn with_capacity(capacity: usize) -> String
```

Okay, so you put a number in and it gives you a `String`. That's easy, but maybe we are curious and want to see more. If you click on [src] you can see this:

```rust
// 🚧
pub fn with_capacity(capacity: usize) -> String {
    String { vec: Vec::with_capacity(capacity) }
}
```

Interesting! Now you can see that a String is a kind of `Vec`. And actually a `String` is a vector of `u8` bytes, which is interesting to know. You didn't need to know that to use the `with_capacity` method so you only see it if you click [src]. So clicking on [src] is a good idea if the document doesn't have much detail and you want to know more.

### Information on traits

The important part of the documentation for a trait is "Required Methods" on the left. If you see Required Methods, it probably means that you have to write the method yourself. For example, for `Iterator` you need to write the `.next()` method. And for `From` you need to write the `.from()` method. But some traits can be implemented with just an **attribute**, like we see in `#[derive(Debug)]`. `Debug` needs the `.fmt()` method, but usually you just use `#[derive(Debug)]` unless you want to do it yourself. That's why the page on `std::fmt::Debug` says that "Generally speaking, you should just derive a Debug implementation."

