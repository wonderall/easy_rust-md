## The ? operator

There is an even shorter way to deal with `Result` (and `Option`), shorter than `match` and even shorter than `if let`. It is called the "question mark operator", and is just `?`. After a function that returns a result, you can add `?`. This will:

- return what is inside the `Result` if it is `Ok`
- pass the error back if it is `Err`

In other words, it does almost everything for you.

We can try this with `.parse()` again. We will write a function called `parse_str` that tries to turn a `&str` into a `i32`. It looks like this:

```rust
use std::num::ParseIntError;

fn parse_str(input: &str) -> Result<i32, ParseIntError> {
    let parsed_number = input.parse::<i32>()?; // Here is the question mark
    Ok(parsed_number)
}

fn main() {}
```

This function takes a `&str`. If it is `Ok`, it gives an `i32` wrapped in `Ok`. If it is an `Err`, it returns a `ParseIntError`. Then we try to parse the number, and add `?`. That means "check if it is an error, and give what is inside the Result if it is okay". If it is not okay, it will return the error and end. But if it is okay, it will go to the next line. On the next line is the number inside of `Ok()`. We need to wrap it in `Ok` because the return is `Result<i32, ParseIntError>`, not `i32`.

Now, we can try out our function. Let's see what it does with a vec of `&str`s.

```rust
fn parse_str(input: &str) -> Result<i32, std::num::ParseIntError> {
    let parsed_number = input.parse::<i32>()?;
    Ok(parsed_number)
}

fn main() {
    let str_vec = vec!["Seven", "8", "9.0", "nice", "6060"];
    for item in str_vec {
        let parsed = parse_str(item);
        println!("{:?}", parsed);
    }
}
```

This prints:

```text
Err(ParseIntError { kind: InvalidDigit })
Ok(8)
Err(ParseIntError { kind: InvalidDigit })
Err(ParseIntError { kind: InvalidDigit })
Ok(6060)
```

How did we find `std::num::ParseIntError`? One easy way is to "ask" the compiler again.

```rust
fn main() {
    let failure = "Not a number".parse::<i32>();
    failure.rbrbrb(); // ⚠️ Compiler: "What is rbrbrb()???"
}
```

The compiler doesn't understand, and says:

```text
error[E0599]: no method named `rbrbrb` found for enum `std::result::Result<i32, std::num::ParseIntError>` in the current scope
 --> src\main.rs:3:13
  |
3 |     failure.rbrbrb();
  |             ^^^^^^ method not found in `std::result::Result<i32, std::num::ParseIntError>`
```

So `std::result::Result<i32, std::num::ParseIntError>` is the signature we need.

We don't need to write `std::result::Result` because `Result` is always "in scope" (in scope = ready to use). Rust does this for all the types we use a lot so we don't have to write `std::result::Result`, `std::collections::Vec`, etc.

We aren't working with things like files yet, so the ? operator doesn't look too useful yet. But here is a useless but quick example that shows how you can use it on a single line. Instead of making an `i32` with `.parse()`, we'll do a lot more. We'll make an `u16`, then turn it to a `String`, then a `u32`, then to a `String` again, and finally to a `i32`.

```rust
use std::num::ParseIntError;

fn parse_str(input: &str) -> Result<i32, ParseIntError> {
    let parsed_number = input.parse::<u16>()?.to_string().parse::<u32>()?.to_string().parse::<i32>()?; // Add a ? each time to check and pass it on
    Ok(parsed_number)
}

fn main() {
    let str_vec = vec!["Seven", "8", "9.0", "nice", "6060"];
    for item in str_vec {
        let parsed = parse_str(item);
        println!("{:?}", parsed);
    }
}
```

This prints the same thing, but this time we handled three `Result`s in a single line. Later on we will do this with files, because they always return `Result`s because many things can go wrong.

Imagine the following: you want to open a file, write to it, and close it. First you need to successfully find the file (that's a `Result`). Then you need to successfully write to it (that's a `Result`). With `?` you can do that on one line.

### When panic and unwrap are good

Rust has a `panic!` macro that you can use to make it panic. It is easy to use:

```rust
fn main() {
    panic!("Time to panic!");
}
```

The message `"Time to panic!"` displays when you run the program: `thread 'main' panicked at 'Time to panic!', src\main.rs:2:3`

You will remember that `src\main.rs` is the directory and file name, and `2:3` are the line and column numbers. With this information, you can find the code and fix it.

`panic!` is a good macro to use to make sure that you know when something changes. For example, this function called `prints_three_things` always prints index [0], [1], and [2] from a vector. It is okay because we always give it a vector with three items:

```rust
fn prints_three_things(vector: Vec<i32>) {
    println!("{}, {}, {}", vector[0], vector[1], vector[2]);
}

fn main() {
    let my_vec = vec![8, 9, 10];
    prints_three_things(my_vec);
}
```

It prints `8, 9, 10` and everything is fine.

But imagine that later on we write more and more code, and forget that `my_vec` can only be three things. Now `my_vec` in this part has six things:

```rust
fn prints_three_things(vector: Vec<i32>) {
  println!("{}, {}, {}", vector[0], vector[1], vector[2]);
}

fn main() {
  let my_vec = vec![8, 9, 10, 10, 55, 99]; // Now my_vec has six things
  prints_three_things(my_vec);
}
```

No error happens, because [0] and [1] and [2] are all inside this longer `Vec`. But what if it was really important to only have three things? We wouldn't know that there was a problem because the program doesn't panic. We should have done this instead:

```rust
fn prints_three_things(vector: Vec<i32>) {
    if vector.len() != 3 {
        panic!("my_vec must always have three items") // will panic if the length is not 3
    }
    println!("{}, {}, {}", vector[0], vector[1], vector[2]);
}

fn main() {
    let my_vec = vec![8, 9, 10];
    prints_three_things(my_vec);
}
```

Now we will know if the vector has six items because it panics as it should:

```rust
    // ⚠️
fn prints_three_things(vector: Vec<i32>) {
    if vector.len() != 3 {
        panic!("my_vec must always have three items")
    }
    println!("{}, {}, {}", vector[0], vector[1], vector[2]);
}

fn main() {
    let my_vec = vec![8, 9, 10, 10, 55, 99];
    prints_three_things(my_vec);
}
```

This gives us `thread 'main' panicked at 'my_vec must always have three items', src\main.rs:8:9`. Thanks to `panic!`, we now remember that `my_vec` should only have three items. So `panic!` is a good macro to create reminders in your code.

There are three other macros that are similar to `panic!` that you use a lot in testing. They are: `assert!`, `assert_eq!`, and `assert_ne!`.

Here is what they mean:

- `assert!()`: if the part inside `()` is not true, the program will panic.
- `assert_eq!()`: the two items inside `()` must be equal.
- `assert_ne!()`: the two items inside `()` must not be equal. (*ne* means not equal)

Some examples:

```rust
fn main() {
    let my_name = "Loki Laufeyson";

    assert!(my_name == "Loki Laufeyson");
    assert_eq!(my_name, "Loki Laufeyson");
    assert_ne!(my_name, "Mithridates");
}
```

This will do nothing, because all three assert macros are okay. (This is what we want)

You can also add a message if you want.

```rust
fn main() {
    let my_name = "Loki Laufeyson";

    assert!(
        my_name == "Loki Laufeyson",
        "{} should be Loki Laufeyson",
        my_name
    );
    assert_eq!(
        my_name, "Loki Laufeyson",
        "{} and Loki Laufeyson should be equal",
        my_name
    );
    assert_ne!(
        my_name, "Mithridates",
        "You entered {}. Input must not equal Mithridates",
        my_name
    );
}
```

These messages will only display if the program panics. So if you run this:

```rust
fn main() {
    let my_name = "Mithridates";

    assert_ne!(
        my_name, "Mithridates",
        "You enter {}. Input must not equal Mithridates",
        my_name
    );
}
```

It will display:

```text
thread 'main' panicked at 'assertion failed: `(left != right)`
  left: `"Mithridates"`,
 right: `"Mithridates"`: You entered Mithridates. Input must not equal Mithridates', src\main.rs:4:5
```

So it is saying "you said that left != right, but left == right". And it displays our message that says `You entered Mithridates. Input must not equal Mithridates`.

`unwrap` is also good when you are writing your program and you want it to crash when there is a problem. Later, when your code is finished it is good to change `unwrap` to something else that won't crash.

You can also use `expect`, which is like `unwrap` but a bit better because you give it your own message. Textbooks usually give this advice: "If you use `.unwrap()` a lot, at least use `.expect()` for better error messages."

This will crash:

```rust
   // ⚠️
fn get_fourth(input: &Vec<i32>) -> i32 {
    let fourth = input.get(3).unwrap();
    *fourth
}

fn main() {
    let my_vec = vec![9, 0, 10];
    let fourth = get_fourth(&my_vec);
}
```

The error message is `thread 'main' panicked at 'called Option::unwrap() on a None value', src\main.rs:7:18`.

Now we write our own message with `expect`:

```rust
   // ⚠️
fn get_fourth(input: &Vec<i32>) -> i32 {
    let fourth = input.get(3).expect("Input vector needs at least 4 items");
    *fourth
}

fn main() {
    let my_vec = vec![9, 0, 10];
    let fourth = get_fourth(&my_vec);
}
```

It crashes again, but the error is better: `thread 'main' panicked at 'Input vector needs at least 4 items', src\main.rs:7:18`. `.expect()` is a little better than `.unwrap()` because of this, but it will still panic on `None`. Now here is an example of a bad practice, a function that tries to unwrap two times. It takes a `Vec<Option<i32>>`, so maybe each part will have a `Some<i32>` or maybe a `None`.

```rust
fn try_two_unwraps(input: Vec<Option<i32>>) {
    println!("Index 0 is: {}", input[0].unwrap());
    println!("Index 1 is: {}", input[1].unwrap());
}

fn main() {
    let vector = vec![None, Some(1000)]; // This vector has a None, so it will panic
    try_two_unwraps(vector);
}
```

The message is: ``thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src\main.rs:2:32``. We're not sure if it was the first `.unwrap()` or the second `.unwrap()` until we check the line. It would be better to check the length and also to not unwrap. But with `.expect()` at least it will be a *little* better. Here it is with `.expect()`:

```rust
fn try_two_unwraps(input: Vec<Option<i32>>) {
    println!("Index 0 is: {}", input[0].expect("The first unwrap had a None!"));
    println!("Index 1 is: {}", input[1].expect("The second unwrap had a None!"));
}

fn main() {
    let vector = vec![None, Some(1000)];
    try_two_unwraps(vector);
}
```

So that is a bit better: `thread 'main' panicked at 'The first unwrap had a None!', src\main.rs:2:32`. We have the line number as well so we can find it.


You can also use `unwrap_or` if you want to always have a value that you want to choose. If you do this it will never panic. That's:

- 1) good because your program won't panic, but
- 2) maybe not good if you want the program to panic if there's a problem.

But usually we don't want our program to panic, so `unwrap_or` is a good method to use.

```rust
fn main() {
    let my_vec = vec![8, 9, 10];

    let fourth = my_vec.get(3).unwrap_or(&0); // If .get doesn't work, we will make the value &0.
                                              // .get returns a reference, so we need &0 and not 0
                                              // You can write "let *fourth" with a * if you want fourth to be
                                              // a 0 and not a &0, but here we just print so it doesn't matter

    println!("{}", fourth);
}
```

This prints `0` because `.unwrap_or(&0)` gives a 0 even if it is a `None`.

