## Control flow
**See this chapter on YouTube: [Part 1](https://youtu.be/UAymDOpv_us) and [Part 2](https://youtu.be/eqysTfiiQZs)**

Control flow means telling your code what to do in different situations. The simplest control flow is `if`.

```rust
fn main() {
    let my_number = 5;
    if my_number == 7 {
        println!("It's seven");
    }
}
```

Also note that you use `==` and not `=`. `==` is to compare, `=` is to *assign* (to give a value). Also note that we wrote `if my_number == 7` and not `if (my_number == 7)`. You don't need brackets with `if` in Rust.

`else if` and `else` give you more control:

```rust
fn main() {
    let my_number = 5;
    if my_number == 7 {
        println!("It's seven");
    } else if my_number == 6 {
        println!("It's six")
    } else {
        println!("It's a different number")
    }
}
```

This prints `It's a different number` because it's not equal to 7 or 6.

You can add more conditions with `&&` (and) and `||` (or).

```rust
fn main() {
    let my_number = 5;
    if my_number % 2 == 1 && my_number > 0 { // % 2 means the number that remains after diving by two
        println!("It's a positive odd number");
    } else if my_number == 6 {
        println!("It's six")
    } else {
        println!("It's a different number")
    }
}
```

This prints `It's a positive odd number` because when you divide it by 2 you have a remainder of 1, and it's greater than 0.


You can see that too much `if`, `else`, and `else if` can be difficult to read. In this case you can use `match` instead, which looks much cleaner. But you must match for every possible result. For example, this will not work:

```rust
fn main() {
    let my_number: u8 = 5;
    match my_number {
        0 => println!("it's zero"),
        1 => println!("it's one"),
        2 => println!("it's two"),
        // ⚠️
    }
}
```

The compiler says:

```text
error[E0004]: non-exhaustive patterns: `3u8..=std::u8::MAX` not covered
 --> src\main.rs:3:11
  |
3 |     match my_number {
  |           ^^^^^^^^^ pattern `3u8..=std::u8::MAX` not covered
```

This means "you told me about 0 to 2, but `u8`s can go up to 255. What about 3? What about 4? What about 5?" And so on. So you can add `_` which means "anything else".

```rust
fn main() {
    let my_number: u8 = 5;
    match my_number {
        0 => println!("it's zero"),
        1 => println!("it's one"),
        2 => println!("it's two"),
        _ => println!("It's some other number"),
    }
}
```

That prints `It's some other number`.

Remember this for match:

- You write `match` and then make a `{}` code block.
- Write the *pattern* on the left and use a `=>` fat arrow to say what to do when it matches.
- Each line is called an "arm".
- Put a comma between the arms (not a semicolon).

You can declare a value with a match:

```rust
fn main() {
    let my_number = 5;
    let second_number = match my_number {
        0 => 0,
        5 => 10,
        _ => 2,
    };
}
```

`second_number` will be 10. Do you see the semicolon at the end? That is because, after the match is over, we actually told the compiler this: `let second_number = 10;`


You can match on more complicated things too. You use a tuple to do it.

```rust
fn main() {
    let sky = "cloudy";
    let temperature = "warm";

    match (sky, temperature) {
        ("cloudy", "cold") => println!("It's dark and unpleasant today"),
        ("clear", "warm") => println!("It's a nice day"),
        ("cloudy", "warm") => println!("It's dark but not bad"),
        _ => println!("Not sure what the weather is."),
    }
}
```

This prints `It's dark but not bad` because it matches "cloudy" and "warm" for `sky` and `temperature`.

You can even put `if` inside of `match`. This is called a "match guard":

```rust
fn main() {
    let children = 5;
    let married = true;

    match (children, married) {
        (children, married) if married == false => println!("Not married with {} children", children),
        (children, married) if children == 0 && married == true => println!("Married but no children"),
        _ => println!("Married? {}. Number of children: {}.", married, children),
    }
}
```

This will print `Married? true. Number of children: 5.`

You can use _ as many times as you want in a match. In this match on colours, we have three but only check one at a time.

```rust
fn match_colours(rbg: (i32, i32, i32)) {
    match rbg {
        (r, _, _) if r < 10 => println!("Not much red"),
        (_, b, _) if b < 10 => println!("Not much blue"),
        (_, _, g) if g < 10 => println!("Not much green"),
        _ => println!("Each colour has at least 10"),
    }
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
Not much blue
Each colour has at least 10
Not much green
```

This also shows how `match` statements work, because in the first example it only printed `Not much blue`. But `first` also has not much green. A `match` statement always stops when it finds a match, and doesn't check the rest. This is a good example of code that compiles well but is not the code you want.

You can make a really big `match` statement to fix it, but it is probably better to use a `for` loop. We will talk about loops soon.

A match has to return the same type. So you can't do this:

```rust
fn main() {
    let my_number = 10;
    let some_variable = match my_number {
        10 => 8,
        _ => "Not ten", // ⚠️
    };
}
```

The compiler tells you that:

```text
error[E0308]: `match` arms have incompatible types
  --> src\main.rs:17:14
   |
15 |       let some_variable = match my_number {
   |  _________________________-
16 | |         10 => 8,
   | |               - this is found to be of type `{integer}`
17 | |         _ => "Not ten",
   | |              ^^^^^^^^^ expected integer, found `&str`
18 | |     };
   | |_____- `match` arms have incompatible types
```

This will also not work, for the same reason:

```rust
fn main() {
    let some_variable = if my_number == 10 { 8 } else { "something else "}; // ⚠️
}
```

But this works, because it's not a `match` so you have a different `let` statement each time:

```rust
fn main() {
    let my_number = 10;

    if my_number == 10 {
        let some_variable = 8;
    } else {
        let some_variable = "Something else";
    }
}
```

You can also use `@` to give a name to the value of a `match` expression, and then you can use it. In this example we match an `i32` input in a function. If it's 4 or 13 we want to use that number in a `println!` statement. Otherwise, we don't need to use it.

```rust
fn match_number(input: i32) {
    match input {
    number @ 4 => println!("{} is an unlucky number in China (sounds close to 死)!", number),
    number @ 13 => println!("{} is unlucky in North America, lucky in Italy! In bocca al lupo!", number),
    _ => println!("Looks like a normal number"),
    }
}

fn main() {
    match_number(50);
    match_number(13);
    match_number(4);
}
```

This prints:

```text
Looks like a normal number
13 is unlucky in North America, lucky in Italy! In bocca al lupo!
4 is an unlucky number in China (sounds close to 死)!
```

