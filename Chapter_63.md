## Taking user input

One easy way to take input from the user is with `std::io::stdin`. This means "standard in", which is the input from the keyboard. With `stdin()` you can get user input, but then you will want to put it in a `&mut String` with `.read_line()`. Here is a simple example of that, but it both works and doesn't work:

```rust
use std::io;

fn main() {
    println!("Please type something, or x to escape:");
    let mut input_string = String::new();

    while input_string != "x" { // This is the part that doesn't work right
        input_string.clear(); // First clear the String. Otherwise it will keep adding to it
        io::stdin().read_line(&mut input_string).unwrap(); // Get the stdin from the user, and put it in read_string
        println!("You wrote {}", input_string);
    }
    println!("See you later!");
}
```

Here is what an output looks like:

```text
Please type something, or x to escape:
something
You wrote something

Something else
You wrote Something else

x
You wrote x

x
You wrote x

x
You wrote x
```

It takes our input and gives it back, and it even knows that we typed `x`. But it doesn't exit the program. The only way to get out is to close the window, or type ctrl and c. Let's change the `{}` to `{:?}` in `println!` to get more information (or you could use `dbg!(&input_string)` if you like that macro). Now it says:

```text
Please type something, or x to escape:
something
You wrote "something\r\n"
Something else
You wrote "Something else\r\n"
x
You wrote "x\r\n"
x
You wrote "x\r\n"
```



This is because the keyboard input is actually not just `something`, it is `something` and the `Enter` key. There is an easy method to fix this called `.trim()`, which removes all the whitespace. Whitespace, by the way, is all [these characters](https://doc.rust-lang.org/reference/whitespace.html):

```text
U+0009 (horizontal tab, '\t')
U+000A (line feed, '\n')
U+000B (vertical tab)
U+000C (form feed)
U+000D (carriage return, '\r')
U+0020 (space, ' ')
U+0085 (next line)
U+200E (left-to-right mark)
U+200F (right-to-left mark)
U+2028 (line separator)
U+2029 (paragraph separator)
```

So that will turn `x\r\n` into just `x`. Now it works:

```rust
use std::io;

fn main() {
    println!("Please type something, or x to escape:");
    let mut input_string = String::new();

    while input_string.trim() != "x" {
        input_string.clear();
        io::stdin().read_line(&mut input_string).unwrap();
        println!("You wrote {}", input_string);
    }
    println!("See you later!");
}
```

Now it will print:

```text
Please type something, or x to escape:
something
You wrote something

Something
You wrote Something

x
You wrote x

See you later!
```



There is another kind of user input called `std::env::Args` (env means environment). `Args` is what the user types when starting the program. There is actually always at least one `Arg` in a program. Let's write a program that only prints them using `std::env::args()` to see what they are.

```rust
fn main() {
    println!("{:?}", std::env::args());
}
```

If we write `cargo run` then it prints something like this:

```text
Args { inner: ["target\\debug\\rust_book.exe"] }
```

Let's give it more input and see what it does. We'll type `cargo run but with some extra words`. It gives us:

```text
Args { inner: ["target\\debug\\rust_book.exe", "but", "with", "some", "extra", "words"] }
```

Interesting. And when we look at [the page for Args](https://doc.rust-lang.org/std/env/struct.Args.html), we see that it implements `IntoIterator`. That means we can do all the things we know about iterators to read and change it. Let's try this:

```rust
use std::env::args;

fn main() {
    let input = args();

    for entry in input {
        println!("You entered: {}", entry);
    }
}
```

Now it says:

```text
You entered: target\debug\rust_book.exe
You entered: but
You entered: with
You entered: some
You entered: extra
You entered: words
```

You can see that the first argument is always the program name, so you will often want to skip it, like this:

```rust
use std::env::args;

fn main() {
    let input = args();

    input.skip(1).for_each(|item| {
        println!("You wrote {}, which in capital letters is {}", item, item.to_uppercase());
    })
}
```

That will print:

```text
You wrote but, which in capital letters is BUT
You wrote with, which in capital letters is WITH
You wrote some, which in capital letters is SOME
You wrote extra, which in capital letters is EXTRA
You wrote words, which in capital letters is WORDS
```

One common use for `Args` is for user settings. You can make sure that the user writes the input you need, and only run the program if it's right. Here's a small program that either makes letters big (capital) or small (lowercase):

```rust
use std::env::args;

enum Letters {
    Capitalize,
    Lowercase,
    Nothing,
}

fn main() {
    let mut changes = Letters::Nothing;
    let input = args().collect::<Vec<_>>();

    if input.len() > 2 {
        match input[1].as_str() {
            "capital" => changes = Letters::Capitalize,
            "lowercase" => changes = Letters::Lowercase,
            _ => {}
        }
    }

    for word in input.iter().skip(2) {
      match changes {
        Letters::Capitalize => println!("{}", word.to_uppercase()),
        Letters::Lowercase => println!("{}", word.to_lowercase()),
        _ => println!("{}", word)
      }
    }
    
}
```

Here are some examples of what it gives:

Input: `cargo run please make capitals`:

```text
make capitals
```

Input: `cargo run capital`:

```text
// Nothing here...
```

Input: `cargo run capital I think I understand now`:

```text
I
THINK
I
UNDERSTAND
NOW
```

Input: `cargo run lowercase Does this work too?`

```text
does
this
work
too?
```



Besides `Args` given by the user, available in `std::env::args()`, there are also `Vars` which are the system variables. Those are the basic settings for the program that the user didn't type in. You can use `std::env::vars()` to see them all as a `(String, String)`. There are very many. For example:

```rust
fn main() {
    for item in std::env::vars() {
        println!("{:?}", item);
    }
}
```

Just doing this shows you all the information about your user session. It will show information like this:

```text
("CARGO", "/playground/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/cargo")
("CARGO_HOME", "/playground/.cargo")
("CARGO_MANIFEST_DIR", "/playground")
("CARGO_PKG_AUTHORS", "The Rust Playground")
("CARGO_PKG_DESCRIPTION", "")
("CARGO_PKG_HOMEPAGE", "")
("CARGO_PKG_NAME", "playground")
("CARGO_PKG_REPOSITORY", "")
("CARGO_PKG_VERSION", "0.0.1")
("CARGO_PKG_VERSION_MAJOR", "0")
("CARGO_PKG_VERSION_MINOR", "0")
("CARGO_PKG_VERSION_PATCH", "1")
("CARGO_PKG_VERSION_PRE", "")
("DEBIAN_FRONTEND", "noninteractive")
("HOME", "/playground")
("HOSTNAME", "f94c15b8134b")
("LD_LIBRARY_PATH", "/playground/target/debug/build/backtrace-sys-3ec4c973f371c302/out:/playground/target/debug/build/libsqlite3-sys-fbddfbb9b241dacb/out:/playground/target/debug/build/ring-cadba5e583648abb/out:/playground/target/debug/deps:/playground/target/debug:/playground/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/x86_64-unknown-linux-gnu/lib:/playground/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib")
("PATH", "/playground/.cargo/bin:/playground/.cargo/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin")
("PLAYGROUND_EDITION", "2018")
("PLAYGROUND_TIMEOUT", "10")
("PWD", "/playground")
("RUSTUP_HOME", "/playground/.rustup")
("RUSTUP_TOOLCHAIN", "stable-x86_64-unknown-linux-gnu")
("RUST_RECURSION_COUNT", "1")
("SHLVL", "1")
("SSL_CERT_DIR", "/usr/lib/ssl/certs")
("SSL_CERT_FILE", "/usr/lib/ssl/certs/ca-certificates.crt")
("USER", "playground")
("_", "/usr/bin/timeout")
```

So if you need this information, `Vars` is what you want.

The easiest way to get a single `Var` is by using the `env!` macro. You just give it the name of the variable, and it will give you a `&str` with the value. It won't work if the variable is spelled wrong or does not exist, so if you aren't sure then use `option_env!` instead. If we write this on the Playground:

```rust
fn main() {
    println!("{}", env!("USER"));
    println!("{}", option_env!("ROOT").unwrap_or("Can't find ROOT"));
    println!("{}", option_env!("CARGO").unwrap_or("Can't find CARGO"));
}
```

then we get the output:

```text
playground
Can't find ROOT
/playground/.rustup/toolchains/stable-x86_64-unknown-linux-gnu/bin/cargo
```

So `option_env!` is always going to be the safer macro. `env!` is better if you actually want the program to crash when you can't find the environment variable.



