## More about printing

In Rust you can print things in almost any way you want. Here are some more things to know about printing.

Adding `\n` will make a new line, and `\t` will make a tab:

```rust
fn main() {
    // Note: this is print!, not println!
    print!("\t Start with a tab\nand move to a new line");
}
```

This prints:

```text
         Start with a tab
and move to a new line
```

Inside `""` you can write over many lines with no problem, but be careful with the spacing:

```rust
fn main() {
    // Note: After the first line you have to start on the far left.
    // If you write directly under println!, it will add the spaces
    println!("Inside quotes
you can write over
many lines
and it will print just fine.");

    println!("If you forget to write
    on the left side, the spaces
    will be added when you print.");
}
```

This prints:

```text
Inside quotes
you can write over
many lines
and it will print just fine.
If you forget to write
    on the left side, the spaces
    will be added when you print.
```

If you want to print characters like `\n` (called "escape characters"), you can add an extra `\`:

```rust
fn main() {
    println!("Here are two escape characters: \\n and \\t");
}
```

This prints:

```text
Here are two escape characters: \n and \t
```

Sometimes you have too many `"` and escape characters, and want Rust to ignore everything. To do this, you can add `r#` to the beginning and `#` to the end.

```rust
fn main() {
    println!("He said, \"You can find the file at c:\\files\\my_documents\\file.txt.\" Then I found the file."); // We used \ five times here
    println!(r#"He said, "You can find the file at c:\files\my_documents\file.txt." Then I found the file."#)
}
```

This prints the same thing, but using `r#` makes it easier for humans to read.

```text
He said, "You can find the file at c:\files\my_documents\file.txt." Then I found the file.
He said, "You can find the file at c:\files\my_documents\file.txt." Then I found the file.
```

If you need to print with a `#` inside, then you can start with `r##` and end with `##`. And if you need more than one, you can add one more # on each side.

Here are four examples:

```rust
fn main() {

    let my_string = "'Ice to see you,' he said."; // single quotes
    let quote_string = r#""Ice to see you," he said."#; // double quotes
    let hashtag_string = r##"The hashtag #IceToSeeYou had become very popular."##; // Has one # so we need at least ##
    let many_hashtags = r####""You don't have to type ### to use a hashtag. You can just use #.""####; // Has three ### so we need at least ####

    println!("{}\n{}\n{}\n{}\n", my_string, quote_string, hashtag_string, many_hashtags);

}
```

This will print:

```text
'Ice to see you,' he said.
"Ice to see you," he said.
The hashtag #IceToSeeYou had become very popular.
"You don't have to type ### to use a hashtag. You can just use #."
```

`r#` has another use: with it you can use a keyword (words like `let`, `fn`, etc.) as a variable name.

```rust
fn main() {
    let r#let = 6; // The variable's name is let
    let mut r#mut = 10; // This variable's name is mut
}
```

`r#` has this function because older versions of Rust had fewer keywords than Rust now. So with `r#` you can avoid mistakes with variable names that were not keywords before.

Or maybe for some reason you *really* need a function to have a name like `return`. Then you can write this:

```rust
fn r#return() -> u8 {
    println!("Here is your number.");
    8
}

fn main() {
    let my_number = r#return();
    println!("{}", my_number);
}
```

This prints:

```text
Here is your number.
8
```

So you probably won't need it, but if you really need to use a keyword for a variable then you can use `r#`.



If you want to print the bytes of a `&str` or a `char`, you can just write `b` before the string. This works for all ASCII characters. These are all the ASCII characters:

```text
☺☻♥♦♣♠♫☼►◄↕‼¶§▬↨↑↓→∟↔▲▼123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\]^_`abcdefghijklmnopqrstuvwxyz{|}~
```

So when you print this:

```rust
fn main() {
    println!("{:?}", b"This will look like numbers");
}
```

Here is the result:

```text
[84, 104, 105, 115, 32, 119, 105, 108, 108, 32, 108, 111, 111, 107, 32, 108, 105, 107, 101, 32, 110, 117, 109, 98, 101, 114, 115]
```

For a `char` this is called a *byte*, and for a `&str` it's called a *byte string*.


You can also put `b` and `r` together if you need to:

```rust
fn main() {
    println!("{:?}", br##"I like to write "#"."##);
}
```

That will print `[73, 32, 108, 105, 107, 101, 32, 116, 111, 32, 119, 114, 105, 116, 101, 32, 34, 35, 34, 46]`.



There is also a Unicode escape that lets you print any Unicode character inside a string: `\u{}`. A hexadecimal number goes inside the `{}` to print it. Here is a short example of how to get the Unicode number, and how to print it again.

```rust
fn main() {
    println!("{:X}", '행' as u32); // Cast char as u32 to get the hexadecimal value
    println!("{:X}", 'H' as u32);
    println!("{:X}", '居' as u32);
    println!("{:X}", 'い' as u32);

    println!("\u{D589}, \u{48}, \u{5C45}, \u{3044}"); // Try printing them with unicode escape \u
}
```



We know that `println!` can print with `{}` (for Display) and `{:?}` (for Debug), plus `{:#?}` for pretty printing. But there are many other ways to print.

For example, if you have a reference, you can use `{:p}` to print the *pointer address*. Pointer address means the location in your computer's memory.

```rust
fn main() {
    let number = 9;
    let number_ref = &number;
    println!("{:p}", number_ref);
}
```

This prints `0xe2bc0ffcfc` or some other address. It might be different every time, depending on where your computer stores it.

Or you can print binary, hexadecimal and octal:

```rust
fn main() {
    let number = 555;
    println!("Binary: {:b}, hexadecimal: {:x}, octal: {:o}", number, number, number);
}
```

This prints `Binary: 1000101011, hexadecimal: 22b, octal: 1053`.

Or you can add numbers to change the order. The first variable will be in index 0, the next in index 1, and so on.

```rust
fn main() {
    let father_name = "Vlad";
    let son_name = "Adrian Fahrenheit";
    let family_name = "Țepeș";
    println!("This is {1} {2}, son of {0} {2}.", father_name, son_name, family_name);
}
```

`father_name` is in position 0, `son_name` is in position 1, and `family_name` is in position 2. So it prints `This is Adrian Fahrenheit Țepeș, son of Vlad Țepeș`.


Maybe you have a very complex string to print with too many variables inside the `{}` curly brackets. Or maybe you need to print a variable more than one time. Then it can help to add names to the `{}`:

```rust
fn main() {
    println!(
        "{city1} is in {country} and {city2} is also in {country},
but {city3} is not in {country}.",
        city1 = "Seoul",
        city2 = "Busan",
        city3 = "Tokyo",
        country = "Korea"
    );
}
```

That will print:

```text
Seoul is in Korea and Busan is also in Korea,
but Tokyo is not in Korea.
```


Very complex printing is also possible in Rust if you want to use it. Here is how to do it:

{variable:padding alignment minimum.maximum}

To understand this, look at the

1) Do you want a variable name? Write that first, like when we wrote {country} above.
(Then add a `:` after it if you want to do more things)
2) Do you want a padding character? For example, 55 with three "padding zeros" looks like 00055.
3) What alignment (left / middle / right) for the padding?
4) Do you want a minimum length? (just write a number)
5) Do you want a maximum length? (write a number with a `.` in front)

For example, if I want to write "a" with five ㅎ characters on the left and five ㅎ characters on the right:

```rust
fn main() {
    let letter = "a";
    println!("{:ㅎ^11}", letter);
}
```

This prints `ㅎㅎㅎㅎㅎaㅎㅎㅎㅎㅎ`. Let's look at 1) to 5) for this to understand how the compiler reads it.

- Do you want a variable name? `{:ㅎ^11}` There is no variable name. There is nothing before `:`.
- Do you want a padding character? `{:ㅎ^11}` Yes. ㅎ comes after the `:` and has a `^`. `<` means padding with the character on the left, `>` means on the right, and `^` means in the middle.
- Do you want a minimum length? `{:ㅎ^11}` Yes: there is an 11 after.
- Do you want a maximum length? `{:ㅎ^11}` No: there is no number with a `.` before.

Here is an example of many types of formatting.

```rust
fn main() {
    let title = "TODAY'S NEWS";
    println!("{:-^30}", title); // no variable name, pad with -, put in centre, 30 characters long
    let bar = "|";
    println!("{: <15}{: >15}", bar, bar); // no variable name, pad with space, 15 characters each, one to the left, one to the right
    let a = "SEOUL";
    let b = "TOKYO";
    println!("{city1:-<15}{city2:->15}", city1 = a, city2 = b); // variable names city1 and city2, pad with -, one to the left, one to the right
}
```

It prints:

```text
---------TODAY'S NEWS---------
|                            |
SEOUL--------------------TOKYO
```

