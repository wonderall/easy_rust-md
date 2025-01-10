## cargo

`rustc` means Rust compiler, and it's what does the actual compiling. A rust file ends with an `.rs`. But most people don't write something like `rustc main.rs` to compile. They use something called `cargo`, which is the main package manager for Rust.

One note about the name: it's called `cargo` because when you put crates together, you get cargo. A crate is a wooden box that you see on ships or trucks, but you remember that every Rust project is also called a crate. Then when you put them together you get the whole cargo.

You can see this when you use cargo to run a project. Let's try something simple with `rand`: we'll just randomly choose between eight letters.

```rust
use rand::seq::SliceRandom; // Use this for .choose over slices

fn main() {

    let my_letters = vec!['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];

    let mut rng = rand::thread_rng();
    for _ in 0..6 {
        print!("{} ", my_letters.choose(&mut rng).unwrap());
    }
}
```

This will print something like `b c g h e a`. But we want to see what `cargo` does first. To use `cargo` and run our program, usually we type `cargo run`. This will build our program and run it for us. But when it starts compiling, it does something like this:

```text
   Compiling getrandom v0.1.14
   Compiling cfg-if v0.1.10
   Compiling ppv-lite86 v0.2.8
   Compiling rand_core v0.5.1
   Compiling rand_chacha v0.2.2
   Compiling rand v0.7.3
   Compiling rust_book v0.1.0 (C:\Users\mithr\OneDrive\Documents\Rust\rust_book)
    Finished dev [unoptimized + debuginfo] target(s) in 13.13s
     Running `C:\Users\mithr\OneDrive\Documents\Rust\rust_book\target\debug\rust_book.exe`
g f c f h b
```

So it looks like it didn't just bring in `rand`, but some others too. That's because we need `rand` for our crate, but `rand` also has some code that needs other crates too. So `cargo` will find all the crates we need and put them together. In our case we only had seven, but on very big projects you may have 200 or more crates to bring in.

This is where you can see the tradeoff for Rust. Rust is extremely fast, because it compiles ahead of time. It does this by looking through the code and looking to see what the code you write actually does. For example, you might write this generic code:

```rust
use std::fmt::Display;

fn print_and_return_thing<T: Display>(input: T) -> T {
    println!("You gave me {} and now I will give it back.", input);
    input
}

fn main() {
    let my_name = print_and_return_thing("Windy");
    let small_number = print_and_return_thing(9.0);
}
```

This function can take anything with `Display`, so we gave it a `&str` and next gave it a `f64` and that is no problem for us. But the compiler doesn't look at generics, because it doesn't want to do anything at runtime. It wants to put together a program that can run by itself as fast as possible. So when it looks at the first part with `"Windy"`, it doesn't see `fn print_and_return_thing<T: Display>(input: T) -> T`. It sees something like `fn print_and_return_thing(input: &str) -> &str`. And next it sees `fn print_and_return_thing(input: f64) -> f64`. All the checking about traits and so on is done during compile time. That's why generics take longer to compile, because it needs to figure them out, and make it concrete.

One more thing: Rust in 2020 is working hard on compile time, because this part takes the longest. Every version of Rust is a little bit faster at compiling, and there are some other plans to speed it up. But in the meantime, here's what you should know:

- `cargo build` will build your program so you can run it
- `cargo run` will build your program and run it
- `cargo build --release` and `cargo run --release` will do the same but in release mode. What's that? Release mode is for when your code is finally done. Then Rust will take even longer to compile, but it does this because it uses everything it knows to make it faster. Release mode is actually a *lot* faster than the regular mode, which is called debug mode. That's because it compiles quicker and has more debug information. The regular `cargo build` is called a "debug build" and `cargo build --release` is called a "release build".
- `cargo check` is a way to check your code. It's like compiling except that it won't actually make your program. This is a good way to check your code a lot because it doesn't take as long as `build` or `run`.

By the way, the `--release` part of the command is called a `flag`. That means extra information in a command.

Some other things you need to know are:

- `cargo new`. You do this to create a new Rust project. After `new`, write the name of the project and `cargo` will make the folder and all the files you need.
- `cargo clean`. When you add crates to `Cargo.toml`, the computer will download all the files it needs and they can take a lot of space. If you don't want them on your computer anymore, type `cargo clean`.

One more thing about the compiler: it only takes the most time when you use `cargo build` or `cargo run` the first time. After that it will remember, and it will compile fast again. But if you use `cargo clean` and then run `cargo build`, it will have to compile slowly one more time.


