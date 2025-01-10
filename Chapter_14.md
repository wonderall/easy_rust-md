## Strings
**[See this chapter on YouTube](https://youtu.be/pSyaGzGg26o)**

Rust has two main types of strings: `String` and `&str`. What is the difference?

- `&str` is a simple string. When you write `let my_variable = "Hello, world!"`, you create a `&str`. A `&str` is very fast.
- `String` is a more complicated string. It is a bit slower, but it has more functions. A `String` is a pointer, with data on the heap.

Also note that `&str` has the `&` in front of it because you need a reference to use a `str`. That's because of the reason we saw above: the stack needs to know the size. So we give it a `&` that it knows the size of, and then it is happy. Also, because you use a `&` to interact with a `str`, you don't own it. But a `String` is an *owned* type. We will soon learn why that is important to know.

Both `&str` and `String` are UTF-8. For example, you can write:

```rust
fn main() {
    let name = "서태지"; // This is a Korean name. No problem, because a &str is UTF-8.
    let other_name = String::from("Adrian Fahrenheit Țepeș"); // Ț and ș are no problem in UTF-8.
}
```

You can see in `String::from("Adrian Fahrenheit Țepeș")` that it is easy to make a `String` from a `&str`. The two types are very closely linked together, even though they are different.

You can even write emojis, thanks to UTF-8.

```rust
fn main() {
    let name = "😂";
    println!("My name is actually {}", name);
}
```

On your computer that will print `My name is actually 😂` unless your command line can't print it. Then it will show `My name is actually �`. But Rust has no problem with emojis or any other Unicode.

Let's look at the reason for using a `&` for `str`s again to make sure we understand.

- `str` is a dynamically sized type (dynamically sized = the size can be different). For example, the names "서태지" and "Adrian Fahrenheit Țepeș" are not the same size:

```rust
fn main() {

    println!("A String is always {:?} bytes. It is Sized.", std::mem::size_of::<String>()); // std::mem::size_of::<Type>() gives you the size in bytes of a type
    println!("And an i8 is always {:?} bytes. It is Sized.", std::mem::size_of::<i8>());
    println!("And an f64 is always {:?} bytes. It is Sized.", std::mem::size_of::<f64>());
    println!("But a &str? It can be anything. '서태지' is {:?} bytes. It is not Sized.", std::mem::size_of_val("서태지")); // std::mem::size_of_val() gives you the size in bytes of a variable
    println!("And 'Adrian Fahrenheit Țepeș' is {:?} bytes. It is not Sized.", std::mem::size_of_val("Adrian Fahrenheit Țepeș"));
}
```

This prints:

```text
A String is always 24 bytes. It is Sized.
And an i8 is always 1 bytes. It is Sized.
And an f64 is always 8 bytes. It is Sized.
But a &str? It can be anything. '서태지' is 9 bytes. It is not Sized.
And 'Adrian Fahrenheit Țepeș' is 25 bytes. It is not Sized.
```

That is why we need a &, because `&` makes a pointer, and Rust knows the size of the pointer. So the pointer goes on the stack. If we wrote `str`, Rust wouldn't know what to do because it doesn't know the size.



There are many ways to make a `String`. Here are some:

- `String::from("This is the string text");` This is a method for String that takes text and creates a String.
- `"This is the string text".to_string()`. This is a method for &str that makes it a String.
- The `format!` macro. This is like `println!` except it creates a String instead of printing. So you can do this:

```rust
fn main() {
    let my_name = "Billybrobby";
    let my_country = "USA";
    let my_home = "Korea";

    let together = format!(
        "I am {} and I come from {} but I live in {}.",
        my_name, my_country, my_home
    );
}
```

Now we have a String named *together*, but did not print it yet.

One other way to make a String is called `.into()` but it is a bit different because `.into()` isn't just for making a `String`. Some types can easily convert to and from another type using `From` and `.into()`. And if you have `From`, then you also have `.into()`. `From` is clearer because you already know the types: you know that `String::from("Some str")` is a `String` from a `&str`. But with `.into()`, sometimes the compiler doesn't know:

```rust
fn main() {
    let my_string = "Try to make this a String".into(); // ⚠️
}
```

Rust doesn't know what type you want, because many types can be made from a `&str`. It says, "I can make a &str into a lot of things. Which one do you want?"

```text
error[E0282]: type annotations needed
 --> src\main.rs:2:9
  |
2 |     let my_string = "Try to make this a String".into();
  |         ^^^^^^^^^ consider giving `my_string` a type
```

So you can do this:

```rust
fn main() {
    let my_string: String = "Try to make this a String".into();
}
```

And now you get a String.

