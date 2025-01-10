## Type aliases

A type alias means "giving a new name to another type". Type aliases are very easy. Usually you use them when you have a very long type and don't want to write it every time. It is also good when you want to give a type a better name that is easy to remember. Here are two examples of type aliases.

Here is a type that is not difficult, but you want to make your code easier to understand for other people (or for you):

```rust
type CharacterVec = Vec<char>;

fn main() {}
```

Here's a type that is very difficult to read:

```rust
// this return type is extremely long
fn returns<'a>(input: &'a Vec<char>) -> std::iter::Take<std::iter::Skip<std::slice::Iter<'a, char>>> {
    input.iter().skip(4).take(5)
}

fn main() {}
```

So you can change it to this:

```rust
type SkipFourTakeFive<'a> = std::iter::Take<std::iter::Skip<std::slice::Iter<'a, char>>>;

fn returns<'a>(input: &'a Vec<char>) -> SkipFourTakeFive {
    input.iter().skip(4).take(5)
}

fn main() {}
```

Of course, you can also import items to make the type shorter:

```rust
use std::iter::{Take, Skip};
use std::slice::Iter;

fn returns<'a>(input: &'a Vec<char>) -> Take<Skip<Iter<'a, char>>> {
    input.iter().skip(4).take(5)
}

fn main() {}
```

So you can decide what looks best in your code depending on what you like.

Note that this doesn't create an actual new type. It's just a name to use instead of an existing type. So if you write `type File = String;`, the compiler just sees a `String`. So this will print `true`:

```rust
type File = String;

fn main() {
    let my_file = File::from("I am file contents");
    let my_string = String::from("I am file contents");
    println!("{}", my_file == my_string);
}
```

So what if you want an actual new type?

If you want a new file type that the compiler sees as a `File`, you can put it in a struct. (This is actually called the `newtype` idiom)

```rust
struct File(String); // File is a wrapper around String

fn main() {
    let my_file = File(String::from("I am file contents"));
    let my_string = String::from("I am file contents");
}
```

Now this will not work, because they are two different types:

```rust
struct File(String); // File is a wrapper around String

fn main() {
    let my_file = File(String::from("I am file contents"));
    let my_string = String::from("I am file contents");
    println!("{}", my_file == my_string);  // ⚠️ cannot compare File with String
}
```

If you want to compare the String inside, you can use my_file.0:

```rust
struct File(String);

fn main() {
    let my_file = File(String::from("I am file contents"));
    let my_string = String::from("I am file contents");
    println!("{}", my_file.0 == my_string); // my_file.0 is a String, so this prints true
}
```

And now this type doesn't have any traits, so you can implement them yourself. This is not too surprising:

```rust
#[derive(Clone, Debug)]
struct File(String);
```

So when you use the `File` type here you can clone it and Debug print it, but it doesn't have the traits of String unless you use `.0` to get to the String inside it. But in other people's code you can only use `.0` if it's marked `pub` for public. And that's why these sorts of types use the `Deref` trait a lot. We will learn about both `pub` and `Deref` later.

### Importing and renaming inside a function

Usually you write `use` at the top of the program, like this:

```rust
use std::cell::{Cell, RefCell};

fn main() {}
```

But we saw that you can do this anywhere, especially in functions with enums that have long names. Here is an example.

```rust
enum MapDirection {
    North,
    NorthEast,
    East,
    SouthEast,
    South,
    SouthWest,
    West,
    NorthWest,
}

fn main() {}

fn give_direction(direction: &MapDirection) {
    match direction {
        MapDirection::North => println!("You are heading north."),
        MapDirection::NorthEast => println!("You are heading northeast."),
        // So much more left to type...
        // ⚠️ because we didn't write every possible variant
    }
}
```

So now we will import MapDirection inside the function. That means that inside the function you can just write `North` and so on.

```rust
enum MapDirection {
    North,
    NorthEast,
    East,
    SouthEast,
    South,
    SouthWest,
    West,
    NorthWest,
}

fn main() {}

fn give_direction(direction: &MapDirection) {
    use MapDirection::*; // Import everything in MapDirection
    let m = "You are heading";

    match direction {
        North => println!("{} north.", m),
        NorthEast => println!("{} northeast.", m),
        // This is a bit better
        // ⚠️
    }
}
```

We've seen that `::*` means "import everything after the ::". In our case, that means `North`, `NorthEast`...and all the way to `NorthWest`. When you import other people's code you can do that too, but if the code is very large you might have problems. What if it has some items that are the same as your code? So it's usually best to not use `::*` all the time unless you're sure. A lot of times you see  a section called `prelude` in other people's code with all the main items you probably need. So then you will usually use it like this: `name::prelude::*`. We will talk about this more in the sections for `modules` and `crates`.

You can also use `as` to change the name. For example, maybe you are using someone else's code and you can't change the names in an enum:

```rust
enum FileState {
    CannotAccessFile,
    FileOpenedAndReady,
    NoSuchFileExists,
    SimilarFileNameInNextDirectory,
}

fn main() {}
```

So then you can 1) import everything and 2) change the names:

```rust
enum FileState {
    CannotAccessFile,
    FileOpenedAndReady,
    NoSuchFileExists,
    SimilarFileNameInNextDirectory,
}

fn give_filestate(input: &FileState) {
    use FileState::{
        CannotAccessFile as NoAccess,
        FileOpenedAndReady as Good,
        NoSuchFileExists as NoFile,
        SimilarFileNameInNextDirectory as OtherDirectory
    };
    match input {
        NoAccess => println!("Can't access file."),
        Good => println!("Here is your file"),
        NoFile => println!("Sorry, there is no file by that name."),
        OtherDirectory => println!("Please check the other directory."),
    }
}

fn main() {}
```

So now you can write `OtherDirectory` instead of `FileState::SimilarFileNameInNextDirectory`.

