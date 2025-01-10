## Mutable references
**[See this chapter on YouTube](https://youtu.be/G48z6Rv76vc)**

If you want to use a reference to change data, you can use a mutable reference. For a mutable reference, you write `&mut` instead of `&`.

```rust
fn main() {
    let mut my_number = 8; // don't forget to write mut here!
    let num_ref = &mut my_number;
}
```

So what are the two types? `my_number` is an `i32`, and `num_ref` is `&mut i32` (we say a "mutable reference to an `i32`").

So let's use it to add 10 to my_number. But you can't write `num_ref += 10`, because `num_ref` is not the `i32` value, it is a `&i32`. The value is actually inside the `i32`. To reach the place where the value is, we use `*`. `*` means "I don't want the reference, I want the value behind the reference". In other words, one `*` is the opposite of `&`. Also, one `*` erases one `&`.

```rust
fn main() {
    let mut my_number = 8;
    let num_ref = &mut my_number;
    *num_ref += 10; // Use * to change the i32 value.
    println!("{}", my_number);

    let second_number = 800;
    let triple_reference = &&&second_number;
    println!("Second_number = triple_reference? {}", second_number == ***triple_reference);
}
```

This prints:

```text
18
Second_number = triple_reference? true
```

Because using `&` is called "referencing", using `*` is called "**de**referencing".

Rust has two rules for mutable and immutable references. They are very important, but also easy to remember because they make sense.

- **Rule 1**: If you have only immutable references, you can have as many as you want. 1 is fine, 3 is fine, 1000 is fine. No problem.
- **Rule 2**: If you have a mutable reference, you can only have one. Also, you can't have an immutable reference **and** a mutable reference together.

This is because mutable references can change the data. You could get problems if you change the data when other references are reading it.


A good way to understand is to think of a Powerpoint presentation.

Situation one is about **only one mutable reference**.

Situation one: An employee is writing a Powerpoint presentation. He wants his manager to help him. The employee gives his login information to his manager, and asks him to help by making edits. Now the manager has a "mutable reference" to the employee's presentation. The manager can make any changes he wants, and give the computer back later. This is fine, because nobody else is looking at the presentation.

Situation two is about **only immutable references**.

Situation two: The employee is giving the presentation to 100 people. All 100 people can now see the employee's data. They all have an "immutable reference" to the employee's presentation. This is fine, because they can see it but nobody can change the data.

Situation three is **the problem situation**.

Situation three: The Employee gives his manager his login information. His manager now has a "mutable reference". Then the employee went to give the presentation to 100 people, but the manager can still login. This is not fine, because the manager can log in and do anything. Maybe his manager will log into the computer and start typing an email to his mother! Now the 100 people have to watch the manager write an email to his mother instead of the presentation. That's not what they expected to see.

Here is an example of a mutable borrow with an immutable borrow:

```rust
fn main() {
    let mut number = 10;
    let number_ref = &number;
    let number_change = &mut number;
    *number_change += 10;
    println!("{}", number_ref); // ⚠️
}
```

The compiler prints a helpful message to show us the problem.

```text
error[E0502]: cannot borrow `number` as mutable because it is also borrowed as immutable
 --> src\main.rs:4:25
  |
3 |     let number_ref = &number;
  |                      ------- immutable borrow occurs here
4 |     let number_change = &mut number;
  |                         ^^^^^^^^^^^ mutable borrow occurs here
5 |     *number_change += 10;
6 |     println!("{}", number_ref);
  |                    ---------- immutable borrow later used here
```

However, this code will work. Why?

```rust
fn main() {
    let mut number = 10;
    let number_change = &mut number; // create a mutable reference
    *number_change += 10; // use mutable reference to add 10
    let number_ref = &number; // create an immutable reference
    println!("{}", number_ref); // print the immutable reference
}
```

It prints `20` with no problem. It works because the compiler is smart enough to understand our code. It knows that we used `number_change` to change `number`, but didn't use it again. So here there is no problem. We are not using immutable and mutable references together.

Earlier in Rust this kind of code actually generated an error, but the compiler is smarter now. It can understand not just what we type, but how we use everything.

### Shadowing again

Remember when we said that shadowing doesn't **destroy** a value but **blocks** it? Now we can use references to see this.

```rust
fn main() {
    let country = String::from("Austria");
    let country_ref = &country;
    let country = 8;
    println!("{}, {}", country_ref, country);
}
```

Does this print `Austria, 8` or `8, 8`? It prints `Austria, 8`. First we declare a `String` called `country`. Then we create a reference `country_ref` to this string. Then we shadow country with 8, which is an `i32`. But the first `country` was not destroyed, so `country_ref` still says "Austria", not "8". Here is the same code with some comments to show how it works:

```rust
fn main() {
    let country = String::from("Austria"); // Now we have a String called country
    let country_ref = &country; // country_ref is a reference to this data. It's not going to change
    let country = 8; // Now we have a variable called country that is an i8. But it has no relation to the other one, or to country_ref
    println!("{}, {}", country_ref, country); // country_ref still refers to the data of String::from("Austria") that we gave it.
}
```

