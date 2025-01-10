## Types

Rust has many types that let you work with numbers, characters, and so on. Some are simple, others are more complicated, and you can even create your own.

### Primitive types
**[See this chapter on YouTube](https://youtu.be/OxTPU5UGMhs)**

Rust has simple types that are called **primitive types** (primitive = very basic). We will start with integers and `char` (characters). Integers are whole numbers with no decimal point. There are two types of integers:

- Signed integers,
- Unsigned integers.

Signed means `+` (plus sign) and `-` (minus sign), so signed integers can be positive (e.g. +8), negative (e.g. -8), or zero. But unsigned integers can only be positive or zero, because they do not have a sign.

The signed integers are: `i8`, `i16`, `i32`, `i64`, `i128`, and `isize`.
The unsigned integers are: `u8`, `u16`, `u32`, `u64`, `u128`, and `usize`.

The number after the i or the u means the number of bits for the number, so numbers with more bits can be larger. 8 bits = one byte, so `i8` is one byte, `i64` is 8 bytes, and so on. Number types with larger sizes can hold larger numbers. For example, a `u8` can hold up to 255, but a `u16` can hold up to 65535. And a `u128` can hold up to 340282366920938463463374607431768211455.

So what is `isize` and `usize`? This means the number of bits on your type of computer. (The number of bits on your computer is called the **architecture** of your computer.) So `isize` and `usize` on a 32-bit computer is like `i32` and `u32`, and `isize` and `usize` on a 64-bit computer is like `i64` and `u64`.

There are many reasons for the different types of integers. One reason is computer performance: a smaller number of bytes is faster to process. For example, the number -10 as an `i8` is `11110110`, but as an `i128` it is `11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111110110`. But here are some other uses:

Characters in Rust are called `char`. Every `char` has a number: the letter `A` is number 65, while the character `友` ("friend" in Chinese) is number 21451. The list of numbers is called "Unicode". Unicode uses smaller numbers for characters that are used more, like A through Z, or digits 0 through 9, or space.

```rust
fn main() {
    let first_letter = 'A';
    let space = ' '; // A space inside ' ' is also a char
    let other_language_char = 'Ꮔ'; // Thanks to Unicode, other languages like Cherokee display just fine too
    let cat_face = '😺'; // Emojis are chars too
}
```

The characters that are used most have numbers less than 256, and they can fit into a `u8`. Remember, a `u8` is 0 plus all the numbers up to 255, for 256 in total. This means that Rust can safely **cast** a `u8` into a `char`, using `as`. ("Cast `u8` as `char`" means "pretend `u8` is a `char`")

Casting with `as` is useful because Rust is very strict. It always needs to know the type, and won't let you use two different types together even if they are both integers. For example, this will not work:

```rust
fn main() { // main() is where Rust programs start to run. Code goes inside {} (curly brackets)

    let my_number = 100; // We didn't write a type of integer,
                         // so Rust chooses i32. Rust always
                         // chooses i32 for integers if you don't
                         // tell it to use a different type

    println!("{}", my_number as char); // ⚠️
}
```

Here is the reason:

```text
error[E0604]: only `u8` can be cast as `char`, not `i32`
 --> src\main.rs:3:20
  |
3 |     println!("{}", my_number as char);
  |                    ^^^^^^^^^^^^^^^^^
```

Fortunately we can easily fix this with `as`. We can't cast `i32` as a `char`, but we can cast an `i32` as a `u8`. And then we can do the same from `u8` to `char`. So in one line we use `as` to make my_number a `u8`, and again to make it a `char`. Now it will compile:

```rust
fn main() {
    let my_number = 100;
    println!("{}", my_number as u8 as char);
}
```

It prints `d` because that is the `char` in place 100.

The easier way, however, is just to tell Rust that `my_number` is a `u8`. Here's how you do it:

```rust
fn main() {
    let my_number: u8 = 100; //  change my_number to my_number: u8
    println!("{}", my_number as char);
}
```

So those are two reasons for all the different number types in Rust. Here is another reason: `usize` is the size that Rust uses for *indexing*. (Indexing means "which item is first", "which item is second", etc.) `usize` is the best size for indexing because:

- An index can't be negative, so it needs to be a number with a u
- It should be big, because sometimes you need to index many things, but
- It can't be a u64 because 32-bit computers can't use u64.

So Rust uses `usize` so that your computer can get the biggest number for indexing that it can read.



Let's learn some more about `char`. You saw that a `char` is always one character, and uses `''` instead of `""`.

All `chars` use 4 bytes of memory, since 4 bytes are enough to hold any kind of character:
- Basic letters and symbols usually need 1 out of 4 bytes: `a b 1 2 + - = $ @`
- Other letters like German Umlauts or accents need 2 out of 4 bytes: `ä ö ü ß è é à ñ`
- Korean, Japanese or Chinese characters need 3 or 4 bytes: `国 안 녕`

When using characters as part of a string, the string is encoded to use the least amount of memory needed for each character.

We can use `.len()` to see this for ourselves:

```rust
fn main() {
    println!("Size of a char: {}", std::mem::size_of::<char>()); // 4 bytes
    println!("Size of string containing 'a': {}", "a".len()); // .len() gives the size of the string in bytes
    println!("Size of string containing 'ß': {}", "ß".len());
    println!("Size of string containing '国': {}", "国".len());
    println!("Size of string containing '𓅱': {}", "𓅱".len());
}
```

This prints:

```text
Size of a char: 4
Size of string containing 'a': 1
Size of string containing 'ß': 2
Size of string containing '国': 3
Size of string containing '𓅱': 4
```

You can see that `a` is one byte, the German `ß` is two, the Japanese `国` is three, and the ancient Egyptian `𓅱` is 4 bytes.

```rust
fn main() {
    let slice = "Hello!";
    println!("Slice is {} bytes.", slice.len());
    let slice2 = "안녕!"; // Korean for "hi"
    println!("Slice2 is {} bytes.", slice2.len());
}
```

This prints:

```text
Slice is 6 bytes.
Slice2 is 7 bytes.
```

`slice` is 6 characters in length and 6 bytes, but `slice2` is 3 characters in length and 7 bytes.

If `.len()` gives the size in bytes, what about the size in characters? We will learn about these methods later, but you can just remember that `.chars().count()` will do it. `.chars().count()` turns what you wrote into characters and then counts how many there are.


```rust
fn main() {
    let slice = "Hello!";
    println!("Slice is {} bytes and also {} characters.", slice.len(), slice.chars().count());
    let slice2 = "안녕!";
    println!("Slice2 is {} bytes but only {} characters.", slice2.len(), slice2.chars().count());
}
```

This prints:

```text
Slice is 6 bytes and also 6 characters.
Slice2 is 7 bytes but only 3 characters.
```

