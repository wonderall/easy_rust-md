## Option and Result

We understand enums and generics now, so we can understand `Option` and `Result`. Rust uses these two enums to make code safer.

We will start with `Option`.

### Option

You use `Option` when you have a value that might exist, or might not exist. When a value exists it is `Some(value)` and when it doesn't it's just `None`, Here is an example of bad code that can be improved with `Option`.

```rust
    // ⚠️
fn take_fifth(value: Vec<i32>) -> i32 {
    value[4]
}

fn main() {
    let new_vec = vec![1, 2];
    let index = take_fifth(new_vec);
}
```

When we run the code, it panics. Here is the message:

```text
thread 'main' panicked at 'index out of bounds: the len is 2 but the index is 4', src\main.rs:34:5
```

Panic means that the program stops before the problem happens. Rust sees that the function wants something impossible, and stops. It "unwinds the stack" (takes the values off the stack) and tells you "sorry, I can't do that".

So now we will change the return type from `i32` to `Option<i32>`. This means "give me a `Some(i32)` if it's there, and give me `None` if it's not". We say that the `i32` is "wrapped" in an `Option`, which means that it's inside an `Option`. You have to do something to get the value out.

```rust
fn take_fifth(value: Vec<i32>) -> Option<i32> {
    if value.len() < 5 { // .len() gives the length of the vec.
                         // It must be at least 5.
        None
    } else {
        Some(value[4])
    }
}

fn main() {
    let new_vec = vec![1, 2];
    let bigger_vec = vec![1, 2, 3, 4, 5];
    println!("{:?}, {:?}", take_fifth(new_vec), take_fifth(bigger_vec));
}
```

This prints `None, Some(5)`. This is good, because now we don't panic anymore. But how do we get the value 5?

We can get the value inside an option with `.unwrap()`, but be careful with `.unwrap()`. It's just like unwrapping a present: maybe there's something good inside, or maybe there's an angry snake inside. You only want to `.unwrap()` if you are sure. If you unwrap a value that is `None`, the program will panic.

```rust
// ⚠️
fn take_fifth(value: Vec<i32>) -> Option<i32> {
    if value.len() < 5 {
        None
    } else {
        Some(value[4])
    }
}

fn main() {
    let new_vec = vec![1, 2];
    let bigger_vec = vec![1, 2, 3, 4, 5];
    println!("{:?}, {:?}",
        take_fifth(new_vec).unwrap(), // this one is None. .unwrap() will panic!
        take_fifth(bigger_vec).unwrap()
    );
}
```

The message is:

```text
thread 'main' panicked at 'called `Option::unwrap()` on a `None` value', src\main.rs:14:9
```

But we don't have to use `.unwrap()`. We can use a `match`. Then we can print the value we have `Some`, and not touch it if we have `None`. For example:

```rust
fn take_fifth(value: Vec<i32>) -> Option<i32> {
    if value.len() < 5 {
        None
    } else {
        Some(value[4])
    }
}

fn handle_option(my_option: Vec<Option<i32>>) {
  for item in my_option {
    match item {
      Some(number) => println!("Found a {}!", number),
      None => println!("Found a None!"),
    }
  }
}

fn main() {
    let new_vec = vec![1, 2];
    let bigger_vec = vec![1, 2, 3, 4, 5];
    let mut option_vec = Vec::new(); // Make a new vec to hold our options
                                     // The vec is type: Vec<Option<i32>>. That means a vec of Option<i32>.

    option_vec.push(take_fifth(new_vec)); // This pushes "None" into the vec
    option_vec.push(take_fifth(bigger_vec)); // This pushes "Some(5)" into the vec

    handle_option(option_vec); // handle_option looks at every option in the vec.
                               // It prints the value if it is Some. It doesn't touch it if it is None.
}
```

This prints:

```text
Found a None!
Found a 5!
```

Because we know generics, we are able to read the code for `Option`. It looks like this:

```rust
enum Option<T> {
    None,
    Some(T),
}

fn main() {}
```

The important point to remember: with `Some`, you have a value of type `T` (any type). Also note that the angle brackets after the `enum` name around `T` is what tells the compiler that it's generic. It has no trait like `Display` or anything to limit it, so it can be anything. But with `None`, you don't have anything.

So in a `match` statement for Option you can't say:

```rust
// 🚧
Some(value) => println!("The value is {}", value),
None(value) => println!("The value is {}", value),
```

because `None` is just `None`.

Of course, there are easier ways to use Option. In this code, we will use a method called `.is_some()` to tell us if it is `Some`. (Yes, there is also a method called `.is_none()`.) In this easier way, we don't need `handle_option()` anymore. We also don't need a vec for the Options.

```rust
fn take_fifth(value: Vec<i32>) -> Option<i32> {
    if value.len() < 5 {
        None
    } else {
        Some(value[4])
    }
}

fn main() {
    let new_vec = vec![1, 2];
    let bigger_vec = vec![1, 2, 3, 4, 5];
    let vec_of_vecs = vec![new_vec, bigger_vec];
    for vec in vec_of_vecs {
        let inside_number = take_fifth(vec);
        if inside_number.is_some() {
            // .is_some() returns true if we get Some, false if we get None
            println!("We got: {}", inside_number.unwrap()); // now it is safe to use .unwrap() because we already checked
        } else {
            println!("We got nothing.");
        }
    }
}
```

This prints:

```text
We got nothing.
We got: 5
```

### Result

Result is similar to Option, but here is the difference:

- Option is about `Some` or `None` (value or no value),
- Result is about `Ok` or `Err` (okay result, or error result).

So `Option` is if you are thinking: "Maybe there will be something, and maybe there won't." But `Result` is if you are thinking: "Maybe it will fail."

To compare, here are the signatures for Option and Result.

```rust
enum Option<T> {
    None,
    Some(T),
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn main() {}
```

So Result has a value inside of `Ok`, and a value inside of `Err`. That is because errors usually contain information that describes the error.

`Result<T, E>` means you need to think of what you want to return for `Ok`, and what you want to return for `Err`. Actually, you can decide anything. Even this is okay:

```rust
fn check_error() -> Result<(), ()> {
    Ok(())
}

fn main() {
    check_error();
}
```

`check_error` says "return `()` if we get `Ok`, and return `()` if we get `Err`". Then we return `Ok` with a `()`.

The compiler gives us an interesting warning:

```text
warning: unused `std::result::Result` that must be used
 --> src\main.rs:6:5
  |
6 |     check_error();
  |     ^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_must_use)]` on by default
  = note: this `Result` may be an `Err` variant, which should be handled
```

This is true: we only returned the `Result` but it could have been an `Err`. So let's handle the error a bit, even though we're still not really doing anything.

```rust
fn give_result(input: i32) -> Result<(), ()> {
    if input % 2 == 0 {
        return Ok(())
    } else {
        return Err(())
    }
}

fn main() {
    if give_result(5).is_ok() {
        println!("It's okay, guys")
    } else {
        println!("It's an error, guys")
    }
}
```

This prints `It's an error, guys`. So we just handled our first error.

Remember, the four methods to easily check are `.is_some()`, `is_none()`, `is_ok()`, and `is_err()`.


Sometimes a function with Result will use a `String` for the `Err` value. This is not the best method to use, but it is a little better than what we've done so far.

```rust
fn check_if_five(number: i32) -> Result<i32, String> {
    match number {
        5 => Ok(number),
        _ => Err("Sorry, the number wasn't five.".to_string()), // This is our error message
    }
}

fn main() {
    let mut result_vec = Vec::new(); // Create a new vec for the results

    for number in 2..7 {
        result_vec.push(check_if_five(number)); // push each result into the vec
    }

    println!("{:?}", result_vec);
}
```

Our vec prints:

```text
[Err("Sorry, the number wasn\'t five."), Err("Sorry, the number wasn\'t five."), Err("Sorry, the number wasn\'t five."), Ok(5),
Err("Sorry, the number wasn\'t five.")]
```

Just like Option, `.unwrap()` on `Err` will panic.

```rust
    // ⚠️
fn main() {
    let error_value: Result<i32, &str> = Err("There was an error"); // Create a Result that is already an Err
    println!("{}", error_value.unwrap()); // Unwrap it
}
```

The program panics, and prints:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: "There was an error"', src\main.rs:30:20
```

This information helps you fix your code. `src\main.rs:30:20` means "inside main.rs in directory src, on line 30 and column 20". So you can go there to look at your code and fix the problem.

You can also create your own error types. Result functions in the standard library and other people's code usually do this. For example, this function from the standard library:

```rust
// 🚧
pub fn from_utf8(vec: Vec<u8>) -> Result<String, FromUtf8Error>
```

This function takes a vector of bytes (`u8`) and tries to make a `String`. So the success case for the Result is a `String` and the error case is `FromUtf8Error`. You can give your error type any name you want.

Using a `match` with `Option` and `Result` sometimes requires a lot of code. For example, the `.get()` method returns an `Option` on a `Vec`.

```rust
fn main() {
    let my_vec = vec![2, 3, 4];
    let get_one = my_vec.get(0); // 0 to get the first number
    let get_two = my_vec.get(10); // Returns None
    println!("{:?}", get_one);
    println!("{:?}", get_two);
}
```

This prints

```text
Some(2)
None
```

So now we can match to get the values. Let's use a range from 0 to 10 to see if it matches the numbers in `my_vec`.

```rust
fn main() {
    let my_vec = vec![2, 3, 4];

    for index in 0..10 {
      match my_vec.get(index) {
        Some(number) => println!("The number is: {}", number),
        None => {}
      }
    }
}
```

This is good, but we don't do anything for `None` because we don't care. Here we can make the code smaller by using `if let`. `if let` means "do something if it matches, and don't do anything if it doesn't". `if let` is when you don't care about matching for everything.

```rust
fn main() {
    let my_vec = vec![2, 3, 4];

    for index in 0..10 {
      if let Some(number) = my_vec.get(index) {
        println!("The number is: {}", number);
      }
    }
}
```

**Important to remember**: `if let Some(number) = my_vec.get(index)` means "if you get `Some(number)` from `my_vec.get(index)`".

Also note: it uses one `=`. It is not a boolean.

`while let` is like a while loop for `if let`. Imagine that we have weather station data like this:

```text
["Berlin", "cloudy", "5", "-7", "78"]
["Athens", "sunny", "not humid", "20", "10", "50"]
```

We want to get the numbers, but not the words. For the numbers, we can use a method called `parse::<i32>()`. `parse()` is the method, and `::<i32>` is the type. It will try to turn the `&str` into an `i32`, and give it to us if it can. It returns a `Result`, because it might not work (like if you wanted it to parse "Billybrobby" - that's not a number).

We will also use `.pop()`. This takes the last item off of the vector.

```rust
fn main() {
    let weather_vec = vec![
        vec!["Berlin", "cloudy", "5", "-7", "78"],
        vec!["Athens", "sunny", "not humid", "20", "10", "50"],
    ];
    for mut city in weather_vec {
        println!("For the city of {}:", city[0]); // In our data, every first item is the city name
        while let Some(information) = city.pop() {
            // This means: keep going until you can't pop anymore
            // When the vector reaches 0 items, it will return None
            // and it will stop.
            if let Ok(number) = information.parse::<i32>() {
                // Try to parse the variable we called information
                // This returns a result. If it's Ok(number), it will print it
                println!("The number is: {}", number);
            }  // We don't write anything here because we do nothing if we get an error. Throw them all away
        }
    }
}
```

This will print:

```text
For the city of Berlin:
The number is: 78
The number is: -7
The number is: 5
For the city of Athens:
The number is: 50
The number is: 10
The number is: 20
```

