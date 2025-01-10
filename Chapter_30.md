## Generics

In functions, you write what type to take as input:

```rust
fn return_number(number: i32) -> i32 {
    println!("Here is your number.");
    number
}

fn main() {
    let number = return_number(5);
}
```

But what if you want to take more than just `i32`? You can use generics for this. Generics means "maybe one type, maybe another type".

For generics, you use angle brackets with the type inside, like this: `<T>` This means "any type you put into the function". Usually, generics uses types with one capital letter (T, U, V, etc.), though you don't have to just use one letter.

This is how you change the function to make it generic:

```rust
fn return_number<T>(number: T) -> T {
    println!("Here is your number.");
    number
}

fn main() {
    let number = return_number(5);
}
```

The important part is the `<T>` after the function name. Without this, Rust will think that T is a concrete (concrete = not generic) type, like `String` or `i8`.

This is easier to understand if we write out a type name. See what happens when we change `T` to `MyType`:

```rust
fn return_number(number: MyType) -> MyType { // ⚠️
    println!("Here is your number.");
    number
}
```

As you can see, `MyType` is concrete, not generic. So we need to write this and so now it works:

```rust
fn return_number<MyType>(number: MyType) -> MyType {
    println!("Here is your number.");
    number
}

fn main() {
    let number = return_number(5);
}
```

So the single letter `T` is for human eyes, but the part after the function name is for the compiler's "eyes". Without it, it's not generic.

Now we will go back to type `T`, because Rust code usually uses `T`.

You will remember that some types in Rust are **Copy**, some are **Clone**, some are **Display**, some are **Debug**, and so on. With **Debug**, we can print with `{:?}`. So now you can see that we have a problem if we want to print `T`:

```rust
fn print_number<T>(number: T) {
    println!("Here is your number: {:?}", number); // ⚠️
}

fn main() {
    print_number(5);
}
```

`print_number` needs **Debug** to print `number`, but is `T` a type with `Debug`? Maybe not. Maybe it doesn't have `#[derive(Debug)]`, who knows. The compiler doesn't know either, so it gives an error:

```text
error[E0277]: `T` doesn't implement `std::fmt::Debug`
  --> src\main.rs:29:43
   |
29 |     println!("Here is your number: {:?}", number);
   |                                           ^^^^^^ `T` cannot be formatted using `{:?}` because it doesn't implement `std::fmt::Debug`
```

T doesn't implement **Debug**. So do we implement Debug for T? No, because we don't know what T is. But we can tell the function: "Don't worry, because any type T for this function will have Debug".

```rust
use std::fmt::Debug; // Debug is located at std::fmt::Debug. So now we can just write 'Debug'.

fn print_number<T: Debug>(number: T) { // <T: Debug> is the important part
    println!("Here is your number: {:?}", number);
}

fn main() {
    print_number(5);
}
```

So now the compiler knows: "Okay, this type T is going to have Debug". Now the code works, because `i32` has Debug. Now we can give it many types: `String`, `&str`, and so on, because they all have Debug.

Now we can create a struct and give it Debug with #[derive(Debug)], so now we can print it too. Our function can take `i32`, the struct Animal, and more:

```rust
use std::fmt::Debug;

#[derive(Debug)]
struct Animal {
    name: String,
    age: u8,
}

fn print_item<T: Debug>(item: T) {
    println!("Here is your item: {:?}", item);
}

fn main() {
    let charlie = Animal {
        name: "Charlie".to_string(),
        age: 1,
    };

    let number = 55;

    print_item(charlie);
    print_item(number);
}
```

This prints:

```text
Here is your item: Animal { name: "Charlie", age: 1 }
Here is your item: 55
```

Sometimes we need more than one type in a generic function. We have to write out each type name, and think about how we want to use it. In this example, we want two types. First we want to print a statement for type T. Printing with `{}` is nicer, so we will require `Display` for `T`.

Next is type U, and the two variables `num_1` and `num_2` have type U (U is some sort of number). We want to compare them, so we need `PartialOrd`. That trait lets us use things like `<`, `>`, `==`, and so on. We want to print them too, so we require `Display` for `U` as well.

```rust
use std::fmt::Display;
use std::cmp::PartialOrd;

fn compare_and_display<T: Display, U: Display + PartialOrd>(statement: T, num_1: U, num_2: U) {
    println!("{}! Is {} greater than {}? {}", statement, num_1, num_2, num_1 > num_2);
}

fn main() {
    compare_and_display("Listen up!", 9, 8);
}
```

This prints `Listen up!! Is 9 greater than 8? true`.

So `fn compare_and_display<T: Display, U: Display + PartialOrd>(statement: T, num_1: U, num_2: U)` says:

- The function name is `compare_and_display`,
- The first type is T, and it is generic. It must be a type that can print with {}.
- The next type is U, and it is generic. It must be a type that can print with {}. Also, it must be a type that can compare (use `>`, `<`, and `==`).

Now we can give `compare_and_display` different types. `statement` can be a `String`, a `&str`, anything with Display.

To make generic functions easier to read, we can also write it like this with `where` right before the code block:

```rust
use std::cmp::PartialOrd;
use std::fmt::Display;

fn compare_and_display<T, U>(statement: T, num_1: U, num_2: U)
where
    T: Display,
    U: Display + PartialOrd,
{
    println!("{}! Is {} greater than {}? {}", statement, num_1, num_2, num_1 > num_2);
}

fn main() {
    compare_and_display("Listen up!", 9, 8);
}
```

Using `where` is a good idea when you have many generic types.

Also note:

- If you have one type T and another type T, they must be the same.
- If you have one type T and another type U, they can be different. But they can also be the same.

For example:

```rust
use std::fmt::Display;

fn say_two<T: Display, U: Display>(statement_1: T, statement_2: U) { // Type T needs Display, type U needs Display
    println!("I have two things to say: {} and {}", statement_1, statement_2);
}

fn main() {

    say_two("Hello there!", String::from("I hate sand.")); // Type T is a &str, but type U is a String.
    say_two(String::from("Where is Padme?"), String::from("Is she all right?")); // Both types are String.
}
```

This prints:

```text
I have two things to say: Hello there! and I hate sand.
I have two things to say: Where is Padme? and Is she all right?
```

