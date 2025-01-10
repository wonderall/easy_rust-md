## impl Trait

`impl Trait` is similar to generics. You remember that generics use a type `T` (or any other name) which then gets decided when the program compiles. First let's look at a concrete type:

```rust
fn gives_higher_i32(one: i32, two: i32) {
    let higher = if one > two { one } else { two };
    println!("{} is higher.", higher);
}

fn main() {
    gives_higher_i32(8, 10);
}
```

This prints: `10 is higher.`.

But this only takes `i32`, so now we will make it generic. We need to compare and we need to print with `{}`, so our type T needs `PartialOrd` and `Display`. Remember, this means "only take types that already have `PartialOrd` and `Display`".

```rust
use std::fmt::Display;

fn gives_higher_i32<T: PartialOrd + Display>(one: T, two: T) {
    let higher = if one > two { one } else { two };
    println!("{} is higher.", higher);
}

fn main() {
    gives_higher_i32(8, 10);
}
```

Now let's look at `impl Trait`, which is similar. Instead of a type `T`, we can bring in a type `impl Trait`. Then it will take in a type that implements that trait. It is almost the same:

```rust
fn prints_it(input: impl Into<String> + std::fmt::Display) { // Takes anything that can turn into a String and has Display
    println!("You can print many things, including {}", input);
}

fn main() {
    let name = "Tuon";
    let string_name = String::from("Tuon");
    prints_it(name);
    prints_it(string_name);
}
```

However, the more interesting part is that we can return `impl Trait`, and that lets us return closures because their function signatures are traits. You can see this in the signatures for methods that have them. For example, this is the signature for `.map()`:

```rust
fn map<B, F>(self, f: F) -> Map<Self, F>     // 🚧
    where
        Self: Sized,
        F: FnMut(Self::Item) -> B,
    {
        Map::new(self, f)
    }
```

`fn map<B, F>(self, f: F)` mean that it takes two generic types. `F` is a function that takes one item from the container implementing `.map()` and `B` is the return type of that function. Then after the `where` we see the trait bounds. ("Trait bound" means "it must have this trait".) One is `Sized`, and the next is the closure signature. It must be an `FnMut`, and do the closure on `Self::Item`, which is the iterator that you give it. Then it returns `B`.

So we can do the same thing to return a closure. To return a closure, use `impl` and then the closure signature. Once you return it, you can use it just like a function. Here is a small example of a function that gives you a closure depending on the text you put in. If you put "double" or "triple" in then it multiplies it by 2 or 3, and otherwise it gives you the same number. Because it's a closure we can do anything we want, so we also print a message.

```rust
fn returns_a_closure(input: &str) -> impl FnMut(i32) -> i32 {
    match input {
        "double" => |mut number| {
            number *= 2;
            println!("Doubling number. Now it is {}", number);
            number
        },
        "triple" => |mut number| {
            number *= 40;
            println!("Tripling number. Now it is {}", number);
            number
        },
        _ => |number| {
            println!("Sorry, it's the same: {}.", number);
            number
        },
    }
}

fn main() {
    let my_number = 10;

    // Make three closures
    let mut doubles = returns_a_closure("double");
    let mut triples = returns_a_closure("triple");
    let mut quadruples = returns_a_closure("quadruple");

    doubles(my_number);
    triples(my_number);
    quadruples(my_number);
}
```

Here is a bit longer example. Let's imagine a game where your character is facing monsters that are stronger at night. We can make an enum called `TimeOfDay` to keep track of the day. Your character is named Simon and has a number called `character_fear`, which is an `f64`. It goes up at night and down during the day. We will make a `change_fear` function that changes his fear, but also does other things like write messages. It could look like this:


```rust
enum TimeOfDay { // just a simple enum
    Dawn,
    Day,
    Sunset,
    Night,
}

fn change_fear(input: TimeOfDay) -> impl FnMut(f64) -> f64 { // The function takes a TimeOfDay. It returns a closure.
                                                             // We use impl FnMut(64) -> f64 to say that it needs to
                                                             // change the value, and also gives the same type back.
    use TimeOfDay::*; // So we only have to write Dawn, Day, Sunset, Night
                      // Instead of TimeOfDay::Dawn, TimeOfDay::Day, etc.
    match input {
        Dawn => |x| { // This is the variable character_fear that we give it later
            println!("The morning sun has vanquished the horrible night. You no longer feel afraid.");
            println!("Your fear is now {}", x * 0.5);
            x * 0.5
        },
        Day => |x| {
            println!("What a nice day. Maybe put your feet up and rest a bit.");
            println!("Your fear is now {}", x * 0.2);
            x * 0.2
        },
        Sunset => |x| {
            println!("The sun is almost down! This is no good.");
            println!("Your fear is now {}", x * 1.4);
            x * 1.4
        },
        Night => |x| {
            println!("What a horrible night to have a curse.");
            println!("Your fear is now {}", x * 5.0);
            x * 5.0
        },
    }
}

fn main() {
    use TimeOfDay::*;
    let mut character_fear = 10.0; // Start Simon with 10

    let mut daytime = change_fear(Day); // Make four closures here to call every time we want to change Simon's fear.
    let mut sunset = change_fear(Sunset);
    let mut night = change_fear(Night);
    let mut morning = change_fear(Dawn);

    character_fear = daytime(character_fear); // Call the closures on Simon's fear. They give a message and change the fear number.
                                              // In real life we would have a Character struct and use it as a method instead,
                                              // like this: character_fear.daytime()
    character_fear = sunset(character_fear);
    character_fear = night(character_fear);
    character_fear = morning(character_fear);
}
```

This prints:

```text
What a nice day. Maybe put your feet up and rest a bit.
Your fear is now 2
The sun is almost down! This is no good.
Your fear is now 2.8
What a horrible night to have a curse.
Your fear is now 14
The morning sun has vanquished the horrible night. You no longer feel afraid.
Your fear is now 7
```

