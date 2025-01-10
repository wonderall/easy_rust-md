## External crates

An external crate means "someone else's crate".

For this section you *almost* need to install Rust, but we can still use just the Playground. Now we are going to learn how to import crates that other people have written. This is important in Rust because of two reasons:

- It is very easy to import other crates, and
- The Rust standard library is quite small.

That means that it is normal in Rust to bring in an external crate for a lot of basic functions. The idea is that if it is easy to use external crates, then you can choose the best one. Maybe one person will make a crate for one function, and then someone else will make a better one.

In this book we will only look at the most popular crates, the crates that everyone who uses Rust knows.

To begin learning external crates, we will start with the most common one: `rand`.

### rand

Did you notice that we didn't use any random numbers yet? That's because random numbers aren't in the standard library. But there are a lot of crates that are "almost standard library" because everybody uses them. In any case, it's very easy to bring in a crate. If you have Rust on your computer, there is a file called `Cargo.toml` that has this information. A `Cargo.toml` file looks like this when you start:

```text
[package]
name = "rust_book"
version = "0.1.0"
authors = ["David MacLeod"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

Now if you want to add the `rand` crate, search for it on `crates.io`, which is where all the crates go. That takes you to `https://crates.io/crates/rand`. And when you click on that, you can see a screen that says `Cargo.toml   rand = "0.7.3"`. All you do is add that under [dependencies] like this:

```text
[package]
name = "rust_book"
version = "0.1.0"
authors = ["David MacLeod"]
edition = "2018"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
rand = "0.7.3"
```

And then Cargo will do the rest for you. Then you can start writing code like [this example code](https://docs.rs/rand/0.7.3/rand/) on the `rand` document website. To get to the documents you can click on the `docs` button in [the page on crates.io](https://crates.io/crates/rand).

So that's enough about Cargo: we are still using just the Playground. Luckily, the Playground already has the top 100 crates installed. So you don't need to write in `Cargo.toml` yet. On the Playground you can imagine that it has a long list like this with 100 crates:

```text
[dependencies]
rand = "0.7.3"
some_other_crate = "0.1.0"
another_nice_crate = "1.7"
```

That means that to use `rand`, you can just do this.

```rust
use rand; // This means the whole crate rand
          // On your computer you can't just write this;
          // you need to write in the Cargo.toml file first

fn main() {
    for _ in 0..5 {
        let random_u16 = rand::random::<u16>();
        print!("{} ", random_u16);
    }
}
```

It will print a different `u16` number every time, like `42266 52873 56528 46927 6867`.


The main functions in `rand` are `random` and `thread_rng` (rng means "random number generator"). And actually if you look at `random` it says: "This is simply a shortcut for `thread_rng().gen()`". So it's actually just `thread_rng` that does almost everything.

Here is a simple example of numbers from 1 to 10. To get those numbers, we use `.gen_range()` between 1 and 11.

```rust
use rand::{thread_rng, Rng}; // Or just use rand::*; if we are lazy

fn main() {
    let mut number_maker = thread_rng();
    for _ in 0..5 {
        print!("{} ", number_maker.gen_range(1, 11));
    }
}
```

This will print something like `7 2 4 8 6`.

With random numbers we can do fun things like make characters for a game. We will use `rand` and some other things we know to make them. In this game our characters have six stats, and you use a d6 for them. A d6 is a cube that gives 1, 2, 3, 4, 5, or 6 when you throw it. Each character rolls a d6 three times, so each stat is between 3 and 18.

But sometimes it can be unfair if your character has something low like a 3 or 4. If your strength is 3 you can't carry anything, for example. So there is one more method that uses a d6 four times. You roll it four times, and throw away the lowest number. So if you roll 3, 3, 1, 6 then you keep 3, 3, 6 = 12. We will make this method too so the owner of the game can decide.

Here is our simple character creator. We created a `Character` struct for the stats, and even implemented `Display` to print it the way we want.

```rust
use rand::{thread_rng, Rng}; // Or just use rand::*; if we are lazy
use std::fmt; // Going to impl Display for our character


struct Character {
    strength: u8,
    dexterity: u8,    // This means "body quickness"
    constitution: u8, // This means "health"
    intelligence: u8,
    wisdom: u8,
    charisma: u8, // This means "popularity with people"
}

fn three_die_six() -> u8 { // A "die" is the thing you throw to get the number
    let mut generator = thread_rng(); // Create our random number generator
    let mut stat = 0; // This is the total
    for _ in 0..3 {
        stat += generator.gen_range(1..=6); // Add each time
    }
    stat // Return the total
}

fn four_die_six() -> u8 {
    let mut generator = thread_rng();
    let mut results = vec![]; // First put the numbers in a vec
    for _ in 0..4 {
        results.push(generator.gen_range(1..=6));
    }
    results.sort(); // Now a result like [4, 3, 2, 6] becomes [2, 3, 4, 6]
    results.remove(0); // Now it would be [3, 4, 6]
    results.iter().sum() // Return this result
}

enum Dice {
    Three,
    Four
}

impl Character {
    fn new(dice: Dice) -> Self { // true for three dice, false for four
        match dice {
            Dice::Three => Self {
                strength: three_die_six(),
                dexterity: three_die_six(),
                constitution: three_die_six(),
                intelligence: three_die_six(),
                wisdom: three_die_six(),
                charisma: three_die_six(),
            },
            Dice::Four => Self {
                strength: four_die_six(),
                dexterity: four_die_six(),
                constitution: four_die_six(),
                intelligence: four_die_six(),
                wisdom: four_die_six(),
                charisma: four_die_six(),
            },
        }
    }
    fn display(&self) { // We can do this because we implemented Display below
        println!("{}", self);
        println!();
    }
}

impl fmt::Display for Character { // Just follow the code for in https://doc.rust-lang.org/std/fmt/trait.Display.html and change it a bit
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f,
            "Your character has these stats:
strength: {}
dexterity: {}
constitution: {}
intelligence: {}
wisdom: {}
charisma: {}",
            self.strength,
            self.dexterity,
            self.constitution,
            self.intelligence,
            self.wisdom,
            self.charisma
        )
    }
}



fn main() {
    let weak_billy = Character::new(Dice::Three);
    let strong_billy = Character::new(Dice::Four);
    weak_billy.display();
    strong_billy.display();
}
```

It will print something like this:

```rust
Your character has these stats:
strength: 9
dexterity: 15
constitution: 15
intelligence: 8
wisdom: 11
charisma: 9

Your character has these stats:
strength: 9
dexterity: 13
constitution: 14
intelligence: 16
wisdom: 16
charisma: 10
```

The character with four dice is usually a bit better at most things.


### rayon

`rayon` is a popular crate that lets you speed up your Rust code. It's popular because it creates threads without needing things like `thread::spawn`. In other words, it is popular because it is effective but easy to write. For example:

- `.iter()`, `.iter_mut()`, `into_iter()` in rayon is written like this:
- `.par_iter()`, `.par_iter_mut()`, `par_into_iter()`. So you just add `par_` and your code becomes much faster. (par means "parallel")

Other methods are the same: `.chars()` is `.par_chars()`, and so on.

Here is an example of a simple piece of code that is making the computer do a lot of work:
```rust
fn main() {
    let mut my_vec = vec![0; 200_000];
    my_vec.iter_mut().enumerate().for_each(|(index, number)| *number+=index+1);
    println!("{:?}", &my_vec[5000..5005]);
}
```

It creates a vector with 200,000 items: each one is 0. Then it calls `.enumerate()` to get the index for each number, and changes the 0 to the index number. It's too long to print so we only print items 5000 to 5004. This is still very fast in Rust, but if you want you can make it faster with Rayon. The code is almost the same:

```rust
use rayon::prelude::*; // Import rayon

fn main() {
    let mut my_vec = vec![0; 200_000];
    my_vec.par_iter_mut().enumerate().for_each(|(index, number)| *number+=index+1); // add par_ to iter_mut
    println!("{:?}", &my_vec[5000..5005]);
}
```

And that's it. `rayon` has many other methods to customize what you want to do, but at its most simple it is just "add `_par` to make your program faster".

### serde

`serde` is a popular crate that lets you convert to and from formats like JSON, YAML, etc. The most common way to use it is by creating a `struct` with two attributes on top. [It looks like this](https://serde.rs/):

```rust
#[derive(Serialize, Deserialize, Debug)]
struct Point {
    x: i32,
    y: i32,
}
```

The `Serialize` and `Deserialize` traits are what make the conversion easy. (That's also where the name `serde` comes from) If you have them on your struct, then you can just call a method to turn it into and from JSON or anything else.

### regex

The [regex](https://crates.io/crates/regex) crate lets you search through text using [regular expressions](https://en.wikipedia.org/wiki/Regular_expression). With that you can get matches for something like `colour`, `color`, `colours` and `colors` through a single search. Regular expressions are a whole other language to learn if you want to use them.

### chrono

[chrono](https://crates.io/crates/chrono) is the main crate for people who need more functionality for time. We will look at the standard library now which has functions for time, but if you need more then this is a good crate to use.


