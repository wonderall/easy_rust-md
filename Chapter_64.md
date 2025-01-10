## Using files

Now that we are using Rust on the computer, we can start working with files. You will notice that now we will start to see more and more `Result`s in our code. That is because once you start working with files and similar things, many things can go wrong. A file might not be there, or maybe the computer can't read it.

You might remember that if you want to use the `?` operator, it has to return a `Result` in the function it is in. If you can't remember the error type, you can just give it nothing and let the compiler tell you. Let's try that with a function that tries to make a number with `.parse()`.

```rust
// ⚠️
fn give_number(input: &str) -> Result<i32, ()> {
    input.parse::<i32>()
}

fn main() {
    println!("{:?}", give_number("88"));
    println!("{:?}", give_number("5"));
}
```

The compiler tells us exactly what to do:

```text
error[E0308]: mismatched types
 --> src\main.rs:4:5
  |
3 | fn give_number(input: &str) -> Result<i32, ()> {
  |                                --------------- expected `std::result::Result<i32, ()>` because of return type
4 |     input.parse::<i32>()
  |     ^^^^^^^^^^^^^^^^^^^^ expected `()`, found struct `std::num::ParseIntError`
  |
  = note: expected enum `std::result::Result<_, ()>`
             found enum `std::result::Result<_, std::num::ParseIntError>`
```

Great! So we just change the return to what the compiler says:

```rust
use std::num::ParseIntError;

fn give_number(input: &str) -> Result<i32, ParseIntError> {
    input.parse::<i32>()
}

fn main() {
    println!("{:?}", give_number("88"));
    println!("{:?}", give_number("5"));
}
```

Now the program works!

```text
Ok(88)
Ok(5)
```

So now we want to use `?` to just give us the value if it works, and the error if it doesn't. But how to do this in `fn main()`? If we try to use `?` in main, it won't work.

```rust
// ⚠️
use std::num::ParseIntError;

fn give_number(input: &str) -> Result<i32, ParseIntError> {
    input.parse::<i32>()
}

fn main() {
    println!("{:?}", give_number("88")?);
    println!("{:?}", give_number("5")?);
}
```

It says:

```text
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `std::ops::Try`)
  --> src\main.rs:8:22
   |
7  | / fn main() {
8  | |     println!("{:?}", give_number("88")?);
   | |                      ^^^^^^^^^^^^^^^^^^ cannot use the `?` operator in a function that returns `()`
9  | |     println!("{:?}", give_number("5")?);
10 | | }
   | |_- this function should return `Result` or `Option` to accept `?`
```

But actually `main()` can return a `Result`, just like any other function. If our function works, we don't want to return anything (main() isn't giving anything to anything else). And if it doesn't work, we will return the same error. So we can write it like this:

```rust
use std::num::ParseIntError;

fn give_number(input: &str) -> Result<i32, ParseIntError> {
    input.parse::<i32>()
}

fn main() -> Result<(), ParseIntError> {
    println!("{:?}", give_number("88")?);
    println!("{:?}", give_number("5")?);
    Ok(())
}
```

Don't forget the `Ok(())` at the end: this is very common in Rust. It means `Ok`, inside of which is `()`, which is our return value. Now it prints:

```text
88
5
```


This wasn't very useful when just using `.parse()`, but it will be with files. That's because `?` also changes error types for us. Here's what [the page for the ? operator](https://doc.rust-lang.org/std/macro.try.html) says in simple English:

```text
If you get an `Err`, it will get the inner error. Then `?` does a conversion using `From`. With that it can change specialized errors to more general ones. The error it gets is then returned.
```

Also, Rust has a convenient `Result` type when using `File`s and similar things. It's called `std::io::Result`, and this is what you usually see in `main()` when you are using `?` to open and do things to files. It's actually a type alias. It looks like this:

```text
type Result<T> = Result<T, Error>;
```

So it is a `Result<T, Error>`, but we only need to write the `Result<T>` part.

Now let's try working with files for the first time. `std::fs` is where the methods are for working with files, and with `std::io::Write` you can write in them. With that we can use `.write_all()` to write into the file.

```rust
use std::fs;
use std::io::Write;

fn main() -> std::io::Result<()> {
    let mut file = fs::File::create("myfilename.txt")?; // Create a file with this name.
                                                        // CAREFUL! If you have a file with this name already,
                                                        // it will delete everything in it.
    file.write_all(b"Let's put this in the file")?;     // Don't forget the b in front of ". That's because files take bytes.
    Ok(())
}
```

Then if you click on the new file `myfilename.txt`, it will say `Let's put this in the file`.

We don't need to do this on two lines though, because we have the `?` operator. It will pass on the result we want if it works, kind of like when you use lots of methods on an iterator. This is when `?` becomes very convenient.

```rust
use std::fs;
use std::io::Write;

fn main() -> std::io::Result<()> {
    fs::File::create("myfilename.txt")?.write_all(b"Let's put this in the file")?;
    Ok(())
}
```

So this is saying "Please try to create a file and check if it worked. If it did, then use `.write_all()` and then check if that worked."

And in fact, there is also a function that does both of these things together. It's called `std::fs::write`. Inside it you give it the file name you want, and the content you want to put inside. Again, careful! It will delete everything in that file if it already exists. Also, it lets you write a `&str` without `b` in front, because of this:

```rust
pub fn write<P: AsRef<Path>, C: AsRef<[u8]>>(path: P, contents: C) -> Result<()>
```

`AsRef<[u8]>` is why you can give it either one.

It's very simple:

```rust
use std::fs;

fn main() -> std::io::Result<()> {
    fs::write("calvin_with_dad.txt", 
"Calvin: Dad, how come old photographs are always black and white? Didn't they have color film back then?
Dad: Sure they did. In fact, those photographs *are* in color. It's just the *world* was black and white then.
Calvin: Really?
Dad: Yep. The world didn't turn color until sometimes in the 1930s...")?;

    Ok(())
}
```

So that's the file we will use. It's a conversation with a comic book character named Calvin and his dad, who is not serious about his question. With this we can create a file to use every time.



Opening a file is just as easy as creating one. You just use `open()` instead of `create()`. After that (if it finds your file), you can do things like `read_to_string()`. To do that you can create a mutable `String` and read the file into there. It looks like this:

```rust
use std::fs;
use std::fs::File;
use std::io::Read; // this is to use the function .read_to_string()

fn main() -> std::io::Result<()> {
     fs::write("calvin_with_dad.txt", 
"Calvin: Dad, how come old photographs are always black and white? Didn't they have color film back then?
Dad: Sure they did. In fact, those photographs *are* in color. It's just the *world* was black and white then.
Calvin: Really?
Dad: Yep. The world didn't turn color until sometimes in the 1930s...")?;


    let mut calvin_file = File::open("calvin_with_dad.txt")?; // Open the file we just made
    let mut calvin_string = String::new(); // This String will hold it
    calvin_file.read_to_string(&mut calvin_string)?; // Read the file into it

    calvin_string.split_whitespace().for_each(|word| print!("{} ", word.to_uppercase())); // Do things with the String now

    Ok(())
}
```

That will print:

```rust
CALVIN: DAD, HOW COME OLD PHOTOGRAPHS ARE ALWAYS BLACK AND WHITE? DIDN'T THEY HAVE COLOR FILM BACK THEN? DAD: SURE THEY DID. IN 
FACT, THOSE PHOTOGRAPHS *ARE* IN COLOR. IT'S JUST THE *WORLD* WAS BLACK AND WHITE THEN. CALVIN: REALLY? DAD: YEP. THE WORLD DIDN'T TURN COLOR UNTIL SOMETIMES IN THE 1930S...
```

Okay, what if we want to create a file but not do it if there is already another file with the same name? Maybe you don't want to delete the other file if it's already there just to make a new one. To do this, there is a struct called `OpenOptions`. Actually, we've been using `OpenOptions` all this time and didn't know it. Take a look at the source for `File::open`:

```rust
pub fn open<P: AsRef<Path>>(path: P) -> io::Result<File> {
        OpenOptions::new().read(true).open(path.as_ref())
    }
```

Interesting, that looks like the builder pattern that we learned. It's the same for `File::create`:

```rust
pub fn create<P: AsRef<Path>>(path: P) -> io::Result<File> {
        OpenOptions::new().write(true).create(true).truncate(true).open(path.as_ref())
    }
```

If you go to [the page for OpenOptions](https://doc.rust-lang.org/std/fs/struct.OpenOptions.html), you can see all the methods that you can choose from. Most take a `bool`:

- `append()`: This means "add to the content that's already there instead of deleting".
- `create()`: This lets `OpenOptions` create a file.
- `create_new()`: This means it will only create a file if it's not there already.
- `read()`: Set this to `true` if you want it to be able to read a file.
- `truncate()`: Set this to true if you want to cut the file content to 0 (delete the contents) when you open it.
- `write()`: This lets it write to a file.

Then at the end you use `.open()` with the file name, and that will give you a `Result`. Let's look at one example:

```rust
// ⚠️
use std::fs;
use std::fs::OpenOptions;

fn main() -> std::io::Result<()> {
     fs::write("calvin_with_dad.txt", 
"Calvin: Dad, how come old photographs are always black and white? Didn't they have color film back then?
Dad: Sure they did. In fact, those photographs *are* in color. It's just the *world* was black and white then.
Calvin: Really?
Dad: Yep. The world didn't turn color until sometimes in the 1930s...")?;

    let calvin_file = OpenOptions::new().write(true).create_new(true).open("calvin_with_dad.txt")?;

    Ok(())
}
```

First we made an `OpenOptions` with `new` (always start with `new`). Then we gave it the ability to `write`. After that we set `create_new()` to `true`, and tried to open the file we made. It won't work, which is what we want:

```text
Error: Os { code: 80, kind: AlreadyExists, message: "The file exists." }
```

Let's try using `.append()` so we can write to a file. To write to the file we can use `.write_all()`, which is a method that tries to write in everything you give it.

Also, we will use the `write!` macro to do the same thing. You will remember this macro from when we did `impl Display` for our structs. This time we are using it on a file though instead of a buffer.

```rust
use std::fs;
use std::fs::OpenOptions;
use std::io::Write;

fn main() -> std::io::Result<()> {
    fs::write("calvin_with_dad.txt", 
"Calvin: Dad, how come old photographs are always black and white? Didn't they have color film back then?
Dad: Sure they did. In fact, those photographs *are* in color. It's just the *world* was black and white then.
Calvin: Really?
Dad: Yep. The world didn't turn color until sometimes in the 1930s...")?;

    let mut calvin_file = OpenOptions::new()
        .append(true) // Now we can write without deleting it
        .read(true)
        .open("calvin_with_dad.txt")?;
    calvin_file.write_all(b"And it was a pretty grainy color for a while too.\n")?;
    write!(&mut calvin_file, "That's really weird.\n")?;
    write!(&mut calvin_file, "Well, truth is stranger than fiction.")?;

    println!("{}", fs::read_to_string("calvin_with_dad.txt")?);

    Ok(())
}
```

This prints:

```text
Calvin: Dad, how come old photographs are always black and white? Didn't they have color film back then?
Dad: Sure they did. In fact, those photographs *are* in color. It's just the *world* was black and white then.
Calvin: Really?
Dad: Yep. The world didn't turn color until sometimes in the 1930s...And it was a pretty grainy color for a while too.
That's really weird.
Well, truth is stranger than fiction.
```

