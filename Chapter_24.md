## Structs
**See this chapter on YouTube: [Part 1](https://youtu.be/W23uQghBOFk) and [Part 2](https://youtu.be/GSVhrjLCuNA)**

With structs, you can create your own type. You will use structs all the time in Rust because they are so convenient. Structs are created with the keyword `struct`. The name of a struct should be in UpperCamelCase (capital letter for each word, no spaces). If you write a struct in all lowercase, the compiler will tell you.

There are three types of structs. One is a "unit struct". Unit means "doesn't have anything". For a unit struct, you just write the name and a semicolon.

```rust
struct FileDirectory;
fn main() {}
```

The next is a tuple struct, or an unnamed struct. It is "unnamed" because you only need to write the types, not the field names. Tuple structs are good when you need a simple struct and don't need to remember names.

```rust
struct Colour(u8, u8, u8);

fn main() {
    let my_colour = Colour(50, 0, 50); // Make a colour out of RGB (red, green, blue)
    println!("The second part of the colour is: {}", my_colour.1);
}
```

This prints `The second part of the colour is: 0`.

The third type is the named struct. This is probably the most common struct. In this struct you declare field names and types inside a `{}` code block. Note that you don't write a semicolon after a named struct, because there is a whole code block after it.

```rust
struct Colour(u8, u8, u8); // Declare the same Colour tuple struct

struct SizeAndColour {
    size: u32,
    colour: Colour, // And we put it in our new named struct
}

fn main() {
    let my_colour = Colour(50, 0, 50);

    let size_and_colour = SizeAndColour {
        size: 150,
        colour: my_colour
    };
}
```

You separate fields by commas in a named struct too. For the last field you can add a comma or not - it's up to you. `SizeAndColour` had a comma after `colour`:

```rust
struct Colour(u8, u8, u8); // Declare the same Colour tuple struct

struct SizeAndColour {
    size: u32,
    colour: Colour, // And we put it in our new named struct
}

fn main() {}
```

but you don't need it. But it can be a good idea to always put a comma, because sometimes you will change the order of the fields:

```rust
struct Colour(u8, u8, u8); // Declare the same Colour tuple struct

struct SizeAndColour {
    size: u32,
    colour: Colour // No comma here
}

fn main() {}
```

Then we decide to change the order...

```rust
struct SizeAndColour {
    colour: Colour // ⚠️ Whoops! Now this doesn't have a comma.
    size: u32,
}

fn main() {}
```

But it is not very important either way so you can choose whether to use a comma or not.


Let's create a `Country` struct to give an example. The `Country` struct has the fields `population`, `capital`, and `leader_name`.

```rust
struct Country {
    population: u32,
    capital: String,
    leader_name: String
}

fn main() {
    let population = 500_000;
    let capital = String::from("Elista");
    let leader_name = String::from("Batu Khasikov");

    let kalmykia = Country {
        population: population,
        capital: capital,
        leader_name: leader_name,
    };
}
```

Did you notice that we wrote the same thing twice? We wrote `population: population`, `capital: capital`, and `leader_name: leader_name`. Actually, you don't need to do that. If the field name and variable name are the same, you don't have to write it twice.

```rust
struct Country {
    population: u32,
    capital: String,
    leader_name: String
}

fn main() {
    let population = 500_000;
    let capital = String::from("Elista");
    let leader_name = String::from("Batu Khasikov");

    let kalmykia = Country {
        population,
        capital,
        leader_name,
    };
}
```

