## cargo doc

You might have noticed that Rust documentation always looks almost the same. On the left side you can see `struct`s and `trait`s, code examples are on the right, etc. This is because you can automatically make documentation just by typing `cargo doc`.

Even making a project with nothing can help you learn about traits in Rust. For example, here are two structs that do almost nothing, and a `fn main()` that also does nothing.

```rust
struct DoesNothing {}
struct PrintThing {}

impl PrintThing {
    fn prints_something() {
        println!("I am printing something");
    }
}

fn main() {}
```


But if you type `cargo doc --open`, you can see a lot more information than you expected. First it shows you this:

```text
Crate rust_book

Structs
DoesNothing
PrintThing

Functions
main
```

But if you click on one of the structs, it will show you a lot of traits that you didn't think were there:

```text
Struct rust_book::DoesNothing
[+] Show declaration
Auto Trait Implementations
impl RefUnwindSafe for DoesNothing
impl Send for DoesNothing
impl Sync for DoesNothing
impl Unpin for DoesNothing
impl UnwindSafe for DoesNothing
Blanket Implementations
impl<T> Any for T
where
    T: 'static + ?Sized,
[src]
[+]
impl<T> Borrow<T> for T
where
    T: ?Sized,
[src]
[+]
impl<T> BorrowMut<T> for T
where
    T: ?Sized,
[src]
[+]
impl<T> From<T> for T
[src]
[+]
impl<T, U> Into<U> for T
where
    U: From<T>,
[src]
[+]
impl<T, U> TryFrom<U> for T
where
    U: Into<T>,
[src]
[+]
impl<T, U> TryInto<U> for T
where
    U: TryFrom<T>,
```

This is because of all the traits that Rust automatically makes for every type.

Then if we add some documentation comments you can see them when you type `cargo doc`.

```rust
/// This is a struct that does nothing
struct DoesNothing {}
/// This struct only has one method.
struct PrintThing {}
/// It just prints the same message.
impl PrintThing {
    fn prints_something() {
        println!("I am printing something");
    }
}

fn main() {}
```


Now it will print:

```text
Crate rust_book
Structs
DoesNothing This is a struct that does nothing
PrintThing  This struct only has one method.
Functions
main
```

`cargo doc` is very nice when you use a lot of other people's crates. Because these crates are all on different websites, it can take some time to search them all. But if you use `cargo doc`, you will have them all in the same place on your hard drive.

