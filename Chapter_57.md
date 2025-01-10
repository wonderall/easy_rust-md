## Crates and modules

Every time you write code in Rust, you are writing it in a `crate`. A `crate` is the file, or files, that go together for your code. Inside the file you write you can also make a `mod`. A `mod` is a space for functions, structs, etc. and is used for a few reasons:

- Building your code: it helps you think about the general structure of your code. This can be important as your code gets larger and larger.
- Reading your code: people can understand your code more easily. For example, the name `std::collections::HashMap` tells you that it's in `std` inside the module `collections`. This gives you a hint that maybe there are more collection types inside `collections` that you can try.
- Privacy: everything starts out as private. That lets you keep users from using functions directly.

To make a `mod`, just write `mod` and start a code block with `{}`. We will make a mod called `print_things` that has some printing-related functions.

```rust
mod print_things {
    use std::fmt::Display;

    fn prints_one_thing<T: Display>(input: T) { // Print anything that implements Display
        println!("{}", input)
    }
}

fn main() {}
```

You can see that we wrote `use std::fmt::Display;` inside `print_things`, because it is a separate space. If you wrote `use std::fmt::Display;` inside `main()` it wouldn't help. Also, we can't call it from `main()` right now. Without the `pub` keyword in front of `fn` it will stay private. Let's try to call it without `pub`. Here's one way to write it:

```rust
// 🚧
fn main() {
    crate::print_things::prints_one_thing(6);
}
```

`crate` means "inside this project", but for our simple example it's the same as "inside this file". Inside that is the mod `print_things`, then finally the `prints_one_thing()` function. You can write that every time, or you can write `use` to import it. Now we can see the error that says that it's private:

```rust
// ⚠️
mod print_things {
    use std::fmt::Display;

    fn prints_one_thing<T: Display>(input: T) {
        println!("{}", input)
    }
}

fn main() {
    use crate::print_things::prints_one_thing;

    prints_one_thing(6);
    prints_one_thing("Trying to print a string...".to_string());
}
```

Here's the error:

```text
error[E0603]: function `prints_one_thing` is private
  --> src\main.rs:10:30
   |
10 |     use crate::print_things::prints_one_thing;
   |                              ^^^^^^^^^^^^^^^^ private function
   |
note: the function `prints_one_thing` is defined here
  --> src\main.rs:4:5
   |
4  |     fn prints_one_thing<T: Display>(input: T) {
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```
It's easy to understand that function `prints_one_thing` is private. It also shows us with `src\main.rs:4:5` where to find the function. This is helpful because you can write `mod`s not just in one file, but over a lot of files as well.

Now we just write `pub fn` instead of `fn` and everything works.

```rust
mod print_things {
    use std::fmt::Display;

    pub fn prints_one_thing<T: Display>(input: T) {
        println!("{}", input)
    }
}

fn main() {
    use crate::print_things::prints_one_thing;

    prints_one_thing(6);
    prints_one_thing("Trying to print a string...".to_string());
}
```

This prints:

```text
6
Trying to print a string...
```

How about `pub` for a struct, enum, trait, or module? `pub` works like this for them:

- `pub` for a struct: it makes the struct public, but the items are not public. To make an item public, you have to write `pub` for each one too.
- `pub` for an enum or trait: everything becomes public. This makes sense because traits are about giving the same behaviour to something. And enums are about choosing between items, and you need to see them all to choose them.
- `pub` for a module: a top level module will be `pub` because if it isn't pub then nobody can touch anything in it at all. But modules inside modules need `pub` to be public.

So let's put a struct named `Billy` inside `print_things`. This struct will be almost all public, but not quite. The struct is public so it will say `pub struct Billy`. Inside it will have a `name` and `times_to_print`. `name` will not be public, because we only want the user to create structs named `"Billy".to_string()`. But the user can select the number of times to print, so that will be public. It looks like this:

```rust
mod print_things {
    use std::fmt::{Display, Debug};

    #[derive(Debug)]
    pub struct Billy { // Billy is public
        name: String, // but name is private.
        pub times_to_print: u32,
    }

    impl Billy {
        pub fn new(times_to_print: u32) -> Self { // That means the user needs to use new to create a Billy. The user can only change the number of times_to_print
            Self {
                name: "Billy".to_string(), // We choose the name - the user can't
                times_to_print,
            }
        }

        pub fn print_billy(&self) { // This function prints a Billy
            for _ in 0..self.times_to_print {
                println!("{:?}", self.name);
            }
        }
    }

    pub fn prints_one_thing<T: Display>(input: T) {
        println!("{}", input)
    }
}

fn main() {
    use crate::print_things::*; // Now we use *. This imports everything from print_things

    let my_billy = Billy::new(3);
    my_billy.print_billy();
}
```

This will print:

```text
"Billy"
"Billy"
"Billy"
```

By the way, the `*` to import everything is called the "glob operator". Glob means "global", so it means everything.

Inside a `mod` you can create other mods. A child mod (a mod inside of a mod) can always use anything inside a parent mod. You can see this in the next example where we have a `mod city` inside a `mod province` inside a `mod country`.

You can think of the structure like this: even if you are in a country, you might not be in a province. And even if you are in a province, you might not be in a city. But if you are in a city, you are in its province and you are in its country.


```rust
mod country { // The main mod doesn't need pub
    fn print_country(country: &str) { // Note: this function isn't public
        println!("We are in the country of {}", country);
    }
    pub mod province { // Make this mod public

        fn print_province(province: &str) { // Note: this function isn't public
            println!("in the province of {}", province);
        }

        pub mod city { // Make this mod public
            pub fn print_city(country: &str, province: &str, city: &str) {  // This function is public though
                crate::country::print_country(country);
                crate::country::province::print_province(province);
                println!("in the city of {}", city);
            }
        }
    }
}

fn main() {
    crate::country::province::city::print_city("Canada", "New Brunswick", "Moncton");
}
```

The interesting part is that `print_city` can access `print_province` and `print_country`. That's because `mod city` is inside the other mods. It doesn't need `pub` in front of `print_province` to use it. And that makes sense: a city doesn't need to do anything to be inside a province and inside a country.

You probably noticed that `crate::country::province::print_province(province);` is very long. When we are inside a module we can use `super` to bring in items from above. Actually the word super itself means "above", like in "superior". In our example we only used the function once, but if you use it more then it is a good idea to import. It can also be a good idea if it makes your code easier to read, even if you only use the function once. The code is almost the same now, but a bit easier to read:

```rust
mod country {
    fn print_country(country: &str) {
        println!("We are in the country of {}", country);
    }
    pub mod province {
        fn print_province(province: &str) {
            println!("in the province of {}", province);
        }

        pub mod city {
            use super::super::*; // use everything in "above above": that means mod country
            use super::*;        // use everything in "above": that means mod province

            pub fn print_city(country: &str, province: &str, city: &str) {
                print_country(country);
                print_province(province);
                println!("in the city of {}", city);
            }
        }
    }
}

fn main() {
    use crate::country::province::city::print_city; // bring in the function

    print_city("Canada", "New Brunswick", "Moncton");
    print_city("Korea", "Gyeonggi-do", "Gwangju"); // Now it's less work to use it again
}
```



