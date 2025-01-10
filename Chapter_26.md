## Loops

With loops you can tell Rust to continue something until you want it to stop. You use `loop` to start a loop that does not stop, unless you tell it when to `break`.

```rust
fn main() { // This program will never stop
    loop {

    }
}
```

So let's tell the compiler when it can break.

```rust
fn main() {
    let mut counter = 0; // set a counter to 0
    loop {
        counter +=1; // increase the counter by 1
        println!("The counter is now: {}", counter);
        if counter == 5 { // stop when counter == 5
            break;
        }
    }
}
```

This will print:

```text
The counter is now: 1
The counter is now: 2
The counter is now: 3
The counter is now: 4
The counter is now: 5
```

If you have a loop inside of a loop, you can give them names. With names, you can tell Rust which loop to `break` out of. Use `'` (called a "tick") and a `:` to give it a name:

```rust
fn main() {
    let mut counter = 0;
    let mut counter2 = 0;
    println!("Now entering the first loop.");

    'first_loop: loop {
        // Give the first loop a name
        counter += 1;
        println!("The counter is now: {}", counter);
        if counter > 9 {
            // Starts a second loop inside this loop
            println!("Now entering the second loop.");

            'second_loop: loop {
                // now we are inside 'second_loop
                println!("The second counter is now: {}", counter2);
                counter2 += 1;
                if counter2 == 3 {
                    break 'first_loop; // Break out of 'first_loop so we can exit the program
                }
            }
        }
    }
}
```

This will print:

```text
Now entering the first loop.
The counter is now: 1
The counter is now: 2
The counter is now: 3
The counter is now: 4
The counter is now: 5
The counter is now: 6
The counter is now: 7
The counter is now: 8
The counter is now: 9
The counter is now: 10
Now entering the second loop.
The second counter is now: 0
The second counter is now: 1
The second counter is now: 2
```

A `while` loop is a loop that continues while something is still `true`. Each loop, Rust will check if it is still `true`. If it becomes `false`, Rust will stop the loop.

```rust
fn main() {
    let mut counter = 0;

    while counter < 5 {
        counter +=1;
        println!("The counter is now: {}", counter);
    }
}
```

A `for` loop lets you tell Rust what to do each time. But in a `for` loop, the loop stops after a certain number of times. `for` loops use **ranges** very often. You use `..` and `..=` to create a range.

- `..` creates an **exclusive** range: `0..3` creates `0, 1, 2`.
- `..=` creates an **inclusive** range: `0..=3` = `0, 1, 2, 3`.

```rust
fn main() {
    for number in 0..3 {
        println!("The number is: {}", number);
    }

    for number in 0..=3 {
        println!("The next number is: {}", number);
    }
}
```

This prints:

```text
The number is: 0
The number is: 1
The number is: 2
The next number is: 0
The next number is: 1
The next number is: 2
The next number is: 3
```

Also notice that `number` becomes the variable name for 0..3. We could have called it `n`, or `ntod_het___hno_f`, or anything. We can then use that name in `println!`.

If you don't need a variable name, use `_`.

```rust
fn main() {
    for _ in 0..3 {
        println!("Printing the same thing three times");
    }
}
```

This prints:

```text
Printing the same thing three times
Printing the same thing three times
Printing the same thing three times
```

because we didn't give it any number to print each time.

And actually, if you give a variable name and don't use it, Rust will tell you:

```rust
fn main() {
    for number in 0..3 {
        println!("Printing the same thing three times");
    }
}
```

This prints the same thing as above. The program compiles fine, but Rust will remind you that you didn't use `number`:

```text
warning: unused variable: `number`
 --> src\main.rs:2:9
  |
2 |     for number in 0..3 {
  |         ^^^^^^ help: if this is intentional, prefix it with an underscore: `_number`
```

Rust suggests writing `_number` instead of `_`. Putting `_` in front of a variable name means "maybe I will use it later". But using just `_` means "I don't care about this variable at all". So you can put `_` in front of variable names if you will use them later and don't want the compiler to tell you about them.

You can also use `break` to return a value. You write the value right after `break` and use a `;`. Here is an example with a `loop` and a break that gives `my_number` its value.

```rust
fn main() {
    let mut counter = 5;
    let my_number = loop {
        counter +=1;
        if counter % 53 == 3 {
            break counter;
        }
    };
    println!("{}", my_number);
}
```

This prints `56`. `break counter;` means "break and return the value of counter". And because the whole block starts with `let`, `my_number` gets the value.

Now that we know how to use loops, here is a better solution to our `match` problem with colours from before. It is a better solution because we want to compare everything, and a `for` loop looks at every item.

```rust
fn match_colours(rbg: (i32, i32, i32)) {
    println!("Comparing a colour with {} red, {} blue, and {} green:", rbg.0, rbg.1, rbg.2);
    let new_vec = vec![(rbg.0, "red"), (rbg.1, "blue"), (rbg.2, "green")]; // Put the colours in a vec. Inside are tuples with the colour names
    let mut all_have_at_least_10 = true; // Start with true. We will set it to false if one colour is less than 10
    for item in new_vec {
        if item.0 < 10 {
            all_have_at_least_10 = false; // Now it's false
            println!("Not much {}.", item.1) // And we print the colour name.
        }
    }
    if all_have_at_least_10 { // Check if it's still true, and print if true
        println!("Each colour has at least 10.")
    }
    println!(); // Add one more line
}

fn main() {
    let first = (200, 0, 0);
    let second = (50, 50, 50);
    let third = (200, 50, 0);

    match_colours(first);
    match_colours(second);
    match_colours(third);
}
```

This prints:

```text
Comparing a colour with 200 red, 0 blue, and 0 green:
Not much blue.
Not much green.

Comparing a colour with 50 red, 50 blue, and 50 green:
Each colour has at least 10.

Comparing a colour with 200 red, 50 blue, and 0 green:
Not much green.
```

