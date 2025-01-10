## Enums
**See this chapter on YouTube: [Part 1](https://youtu.be/SRnqNTJUgjs), [Part 2](https://youtu.be/F_EcbWM63lk), [Part 3](https://youtu.be/2uh64U9JesA) and [Part 4](https://youtu.be/LOHVUYTc5Us)**

An `enum` is short for enumerations. They look very similar to a struct, but are different. Here is the difference:

- Use a `struct` when you want one thing **AND** another thing.
- Use an `enum` when you want one thing **OR** another thing.

So structs are for **many things** together, while enums are for **many choices** together.

To declare an enum, write `enum` and use a code block with the options, separated by commas. Just like a `struct`, the last part can have a comma or not. We will create an enum called `ThingsInTheSky`:

```rust
enum ThingsInTheSky {
    Sun,
    Stars,
}

fn main() {}
```

This is an enum because you can either see the sun, **or** the stars: you have to choose one. These are called **variants**.

```rust
// create the enum with two choices
enum ThingsInTheSky {
    Sun,
    Stars,
}

// With this function we can use an i32 to create ThingsInTheSky.
fn create_skystate(time: i32) -> ThingsInTheSky {
    match time {
        6..=18 => ThingsInTheSky::Sun, // Between 6 and 18 hours we can see the sun
        _ => ThingsInTheSky::Stars, // Otherwise, we can see stars
    }
}

// With this function we can match against the two choices in ThingsInTheSky.
fn check_skystate(state: &ThingsInTheSky) {
    match state {
        ThingsInTheSky::Sun => println!("I can see the sun!"),
        ThingsInTheSky::Stars => println!("I can see the stars!")
    }
}

fn main() {
    let time = 8; // it's 8 o'clock
    let skystate = create_skystate(time); // create_skystate returns a ThingsInTheSky
    check_skystate(&skystate); // Give it a reference so it can read the variable skystate
}
```

This prints `I can see the sun!`.

You can add data to an enum too.

```rust
enum ThingsInTheSky {
    Sun(String), // Now each variant has a string
    Stars(String),
}

fn create_skystate(time: i32) -> ThingsInTheSky {
    match time {
        6..=18 => ThingsInTheSky::Sun(String::from("I can see the sun!")), // Write the strings here
        _ => ThingsInTheSky::Stars(String::from("I can see the stars!")),
    }
}

fn check_skystate(state: &ThingsInTheSky) {
    match state {
        ThingsInTheSky::Sun(description) => println!("{}", description), // Give the string the name description so we can use it
        ThingsInTheSky::Stars(n) => println!("{}", n), // Or you can name it n. Or anything else - it doesn't matter
    }
}

fn main() {
    let time = 8; // it's 8 o'clock
    let skystate = create_skystate(time); // create_skystate returns a ThingsInTheSky
    check_skystate(&skystate); // Give it a reference so it can read the variable skystate
}
```

This prints the same thing: `I can see the sun!`

You can also "import" an enum so you don't have to type so much. Here's an example where we have to type `Mood::` every time we match on our mood:

```rust
enum Mood {
    Happy,
    Sleepy,
    NotBad,
    Angry,
}

fn match_mood(mood: &Mood) -> i32 {
    let happiness_level = match mood {
        Mood::Happy => 10, // Here we type Mood:: every time
        Mood::Sleepy => 6,
        Mood::NotBad => 7,
        Mood::Angry => 2,
    };
    happiness_level
}

fn main() {
    let my_mood = Mood::NotBad;
    let happiness_level = match_mood(&my_mood);
    println!("Out of 1 to 10, my happiness is {}", happiness_level);
}
```

It prints `Out of 1 to 10, my happiness is 7`. Let's import so we can type less. To import everything, write `*`. Note: it's the same key as `*` for dereferencing but is completely different.

```rust
enum Mood {
    Happy,
    Sleepy,
    NotBad,
    Angry,
}

fn match_mood(mood: &Mood) -> i32 {
    use Mood::*; // We imported everything in Mood. Now we can just write Happy, Sleepy, etc.
    let happiness_level = match mood {
        Happy => 10, // We don't have to write Mood:: anymore
        Sleepy => 6,
        NotBad => 7,
        Angry => 2,
    };
    happiness_level
}

fn main() {
    let my_mood = Mood::Happy;
    let happiness_level = match_mood(&my_mood);
    println!("Out of 1 to 10, my happiness is {}", happiness_level);
}
```


Parts of an `enum` can also be turned into an integer. That's because Rust gives each arm of an `enum` a number that starts with 0 for its own use. You can do things with it if your enum doesn't have any other data in it.

```rust
enum Season {
    Spring, // If this was Spring(String) or something it wouldn't work
    Summer,
    Autumn,
    Winter,
}

fn main() {
    use Season::*;
    let four_seasons = vec![Spring, Summer, Autumn, Winter];
    for season in four_seasons {
        println!("{}", season as u32);
    }
}
```

This prints:

```text
0
1
2
3
```

Though you can give it a different number, if you want - Rust doesn't care and can use it in the same way. Just add an `=` and your number to the variant that you want to have a number. You don't have to give all of them a number. But if you don't, Rust will just add 1 from the arm before to give it a number.

```rust
enum Star {
    BrownDwarf = 10,
    RedDwarf = 50,
    YellowStar = 100,
    RedGiant = 1000,
    DeadStar, // Think about this one. What number will it have?
}

fn main() {
    use Star::*;
    let starvec = vec![BrownDwarf, RedDwarf, YellowStar, RedGiant];
    for star in starvec {
        match star as u32 {
            size if size <= 80 => println!("Not the biggest star."), // Remember: size doesn't mean anything. It's just a name we chose so we can print it
            size if size >= 80 => println!("This is a good-sized star."),
            _ => println!("That star is pretty big!"),
        }
    }
    println!("What about DeadStar? It's the number {}.", DeadStar as u32);
}
```

This prints:


```text
Not the biggest star.
Not the biggest star.
This is a good-sized star.
This is a good-sized star.
What about DeadStar? It's the number 1001.
```

`DeadStar` would have been number 4, but now it's 1001.

### Enums to use multiple types

You know that items in a `Vec`, array, etc. all need the same type (only tuples are different). But you can actually use an enum to put different types in. Imagine we want to have a `Vec` with `u32`s or `i32`s. Of course, you can make a `Vec<(u32, i32)>` (a vec with `(u32, i32)` tuples) but we only want one each time. So here you can use an enum. Here is a simple example:

```rust
enum Number {
    U32(u32),
    I32(i32),
}

fn main() {}
```

So there are two variants: the `U32` variant with a `u32` inside, and the `I32` variant with `i32` inside. `U32` and `I32` are just names we made. They could have been `UThirtyTwo` or `IThirtyTwo` or anything else.

Now, if we put them into a `Vec` we just have a `Vec<Number>`, and the compiler is happy because it's all the same type. The compiler doesn't care that we have either `u32` or `i32` because they are all inside a single type called `Number`. And because it's an enum, you have to pick one, which is what we want. We will use the `.is_positive()` method to pick. If it's `true` then we will choose `U32`, and if it's `false` then we will choose `I32`.

Now the code looks like this:

```rust
enum Number {
    U32(u32),
    I32(i32),
}

fn get_number(input: i32) -> Number {
    let number = match input.is_positive() {
        true => Number::U32(input as u32), // change it to u32 if it's positive
        false => Number::I32(input), // otherwise just give the number because it's already i32
    };
    number
}


fn main() {
    let my_vec = vec![get_number(-800), get_number(8)];

    for item in my_vec {
        match item {
            Number::U32(number) => println!("It's a u32 with the value {}", number),
            Number::I32(number) => println!("It's an i32 with the value {}", number),
        }
    }
}
```

This prints what we wanted to see:

```text
It's an i32 with the value -800
It's a u32 with the value 8
```


