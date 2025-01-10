## Copy types

Some types in Rust are very simple. They are called **copy types**. These simple types are all on the stack, and the compiler knows their size. That means that they are very easy to copy, so the compiler always copies when you send it to a function. It always copies because they are so small and easy that there is no reason not to copy. So you don't need to worry about ownership for these types.

These simple types include: integers, floats, booleans (`true` and `false`), and `char`.

How do you know if a type **implements** copy? (implements = can use) You can check the documentation. For example, here is the documentation for char:

[https://doc.rust-lang.org/std/primitive.char.html](https://doc.rust-lang.org/std/primitive.char.html)

On the left you can see **Trait Implementations**. You can see for example **Copy**, **Debug**, and **Display**. So you know that a `char`:

- is copied when you send it to a function (**Copy**)
- can use `{}` to print (**Display**)
- can use `{:?}` to print (**Debug**)

```rust
fn prints_number(number: i32) { // There is no -> so it's not returning anything
                             // If number was not copy type, it would take it
                             // and we couldn't use it again
    println!("{}", number);
}

fn main() {
    let my_number = 8;
    prints_number(my_number); // Prints 8. prints_number gets a copy of my_number
    prints_number(my_number); // Prints 8 again.
                              // No problem, because my_number is copy type!
}
```

But if you look at the documentation for String, it is not copy type.

[https://doc.rust-lang.org/std/string/struct.String.html](https://doc.rust-lang.org/std/string/struct.String.html)

On the left in **Trait Implementations** you can look in alphabetical order. A, B, C... there is no **Copy** in C. But there is **Clone**. **Clone** is similar to **Copy**, but usually needs more memory. Also, you have to call it with `.clone()` - it won't clone just by itself.

In this example, `prints_country()` prints the country name, a `String`. We want to print it two times, but we can't:

```rust
fn prints_country(country_name: String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Kiribati");
    prints_country(country);
    prints_country(country); // ⚠️
}
```

But now we understand the message.

```text
error[E0382]: use of moved value: `country`
 --> src\main.rs:4:20
  |
2 |     let country = String::from("Kiribati");
  |         ------- move occurs because `country` has type `std::string::String`, which does not implement the `Copy` trait
3 |     prints_country(country);
  |                    ------- value moved here
4 |     prints_country(country);
  |                    ^^^^^^^ value used here after move
```

The important part is `which does not implement the Copy trait`. But in the documentation we saw that String implements the `Clone` trait. So we can add `.clone()` to our code. This creates a clone, and we send the clone to the function. Now `country` is still alive, so we can use it.

```rust
fn prints_country(country_name: String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Kiribati");
    prints_country(country.clone()); // make a clone and give it to the function. Only the clone goes in, and country is still alive
    prints_country(country);
}
```

Of course, if the `String` is very large, `.clone()` can use a lot of memory. One `String` can be a whole book in length, and every time we call `.clone()` it will copy the book. So using `&` for a reference is faster, if you can. For example, this code pushes a `&str` onto a `String` and then makes a clone every time it gets used in a function:

```rust
fn get_length(input: String) { // Takes ownership of a String
    println!("It's {} words long.", input.split_whitespace().count()); // splits to count the number of words
}

fn main() {
    let mut my_string = String::new();
    for _ in 0..50 {
        my_string.push_str("Here are some more words "); // push the words on
        get_length(my_string.clone()); // gives it a clone every time
    }
}
```

It prints:

```text
It's 5 words long.
It's 10 words long.
...
It's 250 words long.
```

That's 50 clones. Here it is using a reference instead, which is better:

```rust
fn get_length(input: &String) {
    println!("It's {} words long.", input.split_whitespace().count());
}

fn main() {
    let mut my_string = String::new();
    for _ in 0..50 {
        my_string.push_str("Here are some more words ");
        get_length(&my_string);
    }
}
```

Instead of 50 clones, it's zero.



### Variables without values

A variable without a value is called an "uninitialized" variable. Uninitialized means "hasn't started yet". They are simple: just write `let` and the variable name:

```rust
fn main() {
    let my_variable; // ⚠️
}
```

But you can't use it yet, and Rust won't compile if anything is uninitialized.

But sometimes they can be useful. A good example is when:

- You have a code block and the value for your variable is inside it, and
- The variable needs to live outside of the code block.

```rust
fn loop_then_return(mut counter: i32) -> i32 {
    loop {
        counter += 1;
        if counter % 50 == 0 {
            break;
        }
    }
    counter
}

fn main() {
    let my_number;

    {
        // Pretend we need to have this code block
        let number = {
            // Pretend there is code here to make a number
            // Lots of code, and finally:
            57
        };

        my_number = loop_then_return(number);
    }

    println!("{}", my_number);
}
```

This prints `100`.

You can see that `my_number` was declared in the `main()` function, so it lives until the end. But it gets its value from inside a loop. However, that value lives as long as `my_number`, because `my_number` has the value. And if you wrote `let my_number = loop_then_return(number)` inside the block, it would just die right away.

It helps to imagine if you simplify the code. `loop_then_return(number)` gives the result 100, so let's delete it and write `100` instead. Also, now we don't need `number` so we will delete it too. Now it looks like this:

```rust
fn main() {
    let my_number;
    {
        my_number = 100;
    }

    println!("{}", my_number);
}
```

So it's almost like saying `let my_number = { 100 };`.

Also note that `my_number` is not `mut`. It didn't get a value until it got 100, so it never changed its value. In the end, the real code for `my_number` is just `let my_number = 100;`.

