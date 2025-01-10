## Writing macros

Writing macros can be very complicated. You almost never need to write one, but sometimes you might want to because they are very convenient. Writing macros is interesting because they are almost a different language. To write one, you actually use another macro called `macro_rules!`. Then you add your macro name and open a `{}` block. Inside is sort of like a `match` statement.

Here's one that only takes `()`, then just returns 6:

```rust
macro_rules! give_six {
    () => {
        6
    };
}

fn main() {
    let six = give_six!();
    println!("{}", six);
}
```

But it's not the same as a `match` statement, because a macro actually doesn't compile anything. It just takes an input and gives an output. Then the compiler checks to see if it makes sense. That's why a macro is like "code that writes code". You will remember that a true `match` statement needs to give the same type, so this won't work:

```rust
fn main() {
// ⚠️
    let my_number = 10;
    match my_number {
        10 => println!("You got a ten"),
        _ => 10,
    }
}
```

It will complain that you want to return `()` in one case, and `i32` in the other.

```text
error[E0308]: `match` arms have incompatible types
 --> src\main.rs:5:14
  |
3 | /     match my_number {
4 | |         10 => println!("You got a ten"),
  | |               ------------------------- this is found to be of type `()`
5 | |         _ => 10,
  | |              ^^ expected `()`, found integer
6 | |     }
  | |_____- `match` arms have incompatible types
```

But a macro doesn't care, because it's just giving an output. It's not a compiler - it's code before code. So you can do this:

```rust
macro_rules! six_or_print {
    (6) => {
        6
    };
    () => {
        println!("You didn't give me 6.");
    };
}

fn main() {
    let my_number = six_or_print!(6);
    six_or_print!();
}
```

This is just fine, and prints `You didn't give me 6.`. You can also see that it's not a match arm because there's no `_` case. We can only give it `(6)`, or `()`. Anything else will make an error. And the `6` we give it isn't even an `i32`, it's just an input 6. You can actually set anything as the input for a macro, because it's just looking at input to see what it gets. For example:

```rust
macro_rules! might_print {
    (THis is strange input 하하はは哈哈 but it still works) => {
        println!("You guessed the secret message!")
    };
    () => {
        println!("You didn't guess it");
    };
}

fn main() {
    might_print!(THis is strange input 하하はは哈哈 but it still works);
    might_print!();
}
```

So this strange macro only responds to two things: `()` and `(THis is strange input 하하はは哈哈 but it still works)`. Nothing else. It prints:

```text
You guessed the secret message!
You didn't guess it
```

So a macro isn't exactly Rust syntax. But a macro can also understand different types of input that you give it. Take this example:

```rust
macro_rules! might_print {
    ($input:expr) => {
        println!("You gave me: {}", $input);
    }
}

fn main() {
    might_print!(6);
}
```

This will print `You gave me: 6`. The `$input:expr` part is important. It means "for an expression, give it the variable name $input". In macros, variables start with a `$`. In this macro, if you give it one expression, it will print it. Let's try it out some more:

```rust
macro_rules! might_print {
    ($input:expr) => {
        println!("You gave me: {:?}", $input); // Now we'll use {:?} because we will give it different kinds of expressions
    }
}

fn main() {
    might_print!(()); // give it a ()
    might_print!(6); // give it a 6
    might_print!(vec![8, 9, 7, 10]); // give it a vec
}
```

This will print:

```text
You gave me: ()
You gave me: 6
You gave me: [8, 9, 7, 10]
```

Also note that we wrote `{:?}`, but it won't check to see if `&input` implements `Debug`. It'll just write the code and try to make it compile, and if it doesn't then it gives an error.

So what can a macro see besides `expr`? They are: `block | expr | ident | item | lifetime | literal  | meta | pat | path | stmt | tt | ty | vis`. This is the complicated part. You can see what each of them means [here](https://doc.rust-lang.org/beta/reference/macros-by-example.html), where it says:

```text
item: an Item
block: a BlockExpression
stmt: a Statement without the trailing semicolon (except for item statements that require semicolons)
pat: a Pattern
expr: an Expression
ty: a Type
ident: an IDENTIFIER_OR_KEYWORD
path: a TypePath style path
tt: a TokenTree (a single token or tokens in matching delimiters (), [], or {})
meta: an Attr, the contents of an attribute
lifetime: a LIFETIME_TOKEN
vis: a possibly empty Visibility qualifier
literal: matches -?LiteralExpression
```

There is another good site called cheats.rs that explains them [here](https://cheats.rs/#macros-attributes) and gives examples for each.

However, for most macros you will use `expr`, `ident`, and `tt`. `ident` means identifier and is for variable or function names. `tt` means token tree and sort of means any type of input. Let's try a simple macro with both.

```rust
macro_rules! check {
    ($input1:ident, $input2:expr) => {
        println!(
            "Is {:?} equal to {:?}? {:?}",
            $input1,
            $input2,
            $input1 == $input2
        );
    };
}

fn main() {
    let x = 6;
    let my_vec = vec![7, 8, 9];
    check!(x, 6);
    check!(my_vec, vec![7, 8, 9]);
    check!(x, 10);
}
```

So this will take one `ident` (like a variable name) and an expression and see if they are the same. It prints:

```text
Is 6 equal to 6? true
Is [7, 8, 9] equal to [7, 8, 9]? true
Is 6 equal to 10? false
```

And here's one macro that takes a `tt` and prints it. It uses a macro called `stringify!` to make a string first.

```rust
macro_rules! print_anything {
    ($input:tt) => {
        let output = stringify!($input);
        println!("{}", output);
    };
}

fn main() {
    print_anything!(ththdoetd);
    print_anything!(87575oehq75onth);
}
```

This prints:

```text
ththdoetd
87575oehq75onth
```

But it won't print if we give it something with spaces, commas, etc. It will think that we are giving it more than one item or extra information, so it will be confused.

This is where macros start to get difficult.

To give a macro more than one item at a time, we have to use a different syntax. Instead of `$input`, it will be `$($input1),*`. This means zero or more (this is what * means), separated by a comma. If you want one or more, use `+` instead of `*`.

Now our macro looks like this:

```rust
macro_rules! print_anything {
    ($($input1:tt),*) => {
        let output = stringify!($($input1),*);
        println!("{}", output);
    };
}


fn main() {
    print_anything!(ththdoetd, rcofe);
    print_anything!();
    print_anything!(87575oehq75onth, ntohe, 987987o, 097);
}
```

So it takes any token tree separated by commas, and uses `stringify!` to make it into a string. Then it prints it. It prints:

```text
ththdoetd, rcofe

87575oehq75onth, ntohe, 987987o, 097
```

If we used `+` instead of `*` it would give an error, because one time we gave it no input. So `*` is a bit safer option.

So now we can start to see the power of macros. In this next example we can actually make our own functions:

```rust
macro_rules! make_a_function {
    ($name:ident, $($input:tt),*) => { // First you give it one name for the function, then it checks everything else
        fn $name() {
            let output = stringify!($($input),*); // It makes everything else into a string
            println!("{}", output);
        }
    };
}


fn main() {
    make_a_function!(print_it, 5, 5, 6, I); // We want a function called print_it() that prints everything else we give it
    print_it();
    make_a_function!(say_its_nice, this, is, really, nice); // Same here but we change the function name
    say_its_nice();
}
```

This prints:

```text
5, 5, 6, I
this, is, really, nice
```


So now we can start to understand other macros. You can see that some of the macros we've already been using are pretty simple. Here's the one for `write!` that we used to write to files:

```rust
macro_rules! write {
    ($dst:expr, $($arg:tt)*) => ($dst.write_fmt($crate::format_args!($($arg)*)))
}
```

So to use it, you enter this:

- an expression (`expr`) that gets the variable name `$dst`.
- everything after that. If it wrote `$arg:tt` then it would only take one, but because it wrote `$($arg:tt)*` it takes zero, one, or any number.

Then it takes `$dst` and uses a method called `write_fmt` on it. Inside that, it uses another macro called `format_args!` that takes all `$($arg)*`, or all the arguments we put in.



Now let's take a look at the `todo!` macro. That's the one you use when you want the program to compile but haven't written your code yet. It looks like this:

```rust
macro_rules! todo {
    () => (panic!("not yet implemented"));
    ($($arg:tt)+) => (panic!("not yet implemented: {}", $crate::format_args!($($arg)+)));
}
```

This one has two options: you can enter `()`, or a number of token trees (`tt`).

- If you enter `()`, it just uses `panic!` with a message. So you could actually just write `panic!("not yet implemented")` instead of `todo!` and it would be the same.
- If you enter some arguments, it will try to print them. You can see the same `format_args!` macro inside, which works like `println!`.

So if you write this, it will work too:

```rust
fn not_done() {
    let time = 8;
    let reason = "lack of time";
    todo!("Not done yet because of {}. Check back in {} hours", reason, time);
}

fn main() {
    not_done();
}
```

This will print:

```text
thread 'main' panicked at 'not yet implemented: Not done yet because of lack of time. Check back in 8 hours', src/main.rs:4:5
```


Inside a macro you can even call the same macro. Here's one:

```rust
macro_rules! my_macro {
    () => {
        println!("Let's print this.");
    };
    ($input:expr) => {
        my_macro!();
    };
    ($($input:expr),*) => {
        my_macro!();
    }
}

fn main() {
    my_macro!(vec![8, 9, 0]);
    my_macro!(toheteh);
    my_macro!(8, 7, 0, 10);
    my_macro!();
}
```

This one takes either `()`, or one expression, or many expressions. But it ignores all the expressions no matter what you put in, and just calls `my_macro!` on `()`. So the output is just `Let's print this`, four times.

You can see the same thing in the `dbg!` macro, which also calls itself.

```rust
macro_rules! dbg {
    () => {
        $crate::eprintln!("[{}:{}]", $crate::file!(), $crate::line!()); //$crate means the crate that it's in.
    };
    ($val:expr) => {
        // Use of `match` here is intentional because it affects the lifetimes
        // of temporaries - https://stackoverflow.com/a/48732525/1063961
        match $val {
            tmp => {
                $crate::eprintln!("[{}:{}] {} = {:#?}",
                    $crate::file!(), $crate::line!(), $crate::stringify!($val), &tmp);
                tmp
            }
        }
    };
    // Trailing comma with single argument is ignored
    ($val:expr,) => { $crate::dbg!($val) };
    ($($val:expr),+ $(,)?) => {
        ($($crate::dbg!($val)),+,)
    };
}
```

(`eprintln!` is the same as `println!` except it prints to `io::stderr` instead of `io::stdout`. There is also `eprint!` that doesn't add a new line)

So we can try this out ourself.

```rust
fn main() {
    dbg!();
}
```

That matches the first arm, so it will print the file name and line name with the `file!` and `line!` macros. It prints `[src/main.rs:2]`.

Let's try it with this:

```rust
fn main() {
    dbg!(vec![8, 9, 10]);
}
```

This will match the next arm, because it's one expression. It will then call the input `tmp` and use this code: ` $crate::eprintln!("[{}:{}] {} = {:#?}", $crate::file!(), $crate::line!(), $crate::stringify!($val), &tmp);`. So it will print with `file!` and `line!`, then `$val` made into a `String`, and pretty print with `{:#?}` for `tmp`. So for our input it will write this:

```text
[src/main.rs:2] vec![8, 9, 10] = [
    8,
    9,
    10,
]
```

And for the rest of it it just calls `dbg!` on itself even if you put in an extra comma.

As you can see, macros are very complicated! Usually you only want a macro to automatically do something that a simple function can't do very well. The best way to learn about macros is to look at other macro examples. Not many people can quickly write macros without problems. So don't think that you need to know everything about macros to know how to write in Rust. But if you read other macros, and change them a little, you can easily borrow their power. Then you might start to get comfortable with writing your own.


# Part 2 - Rust on your computer

You saw that we can learn almost anything in Rust just using the Playground. But if you learned everything so far, you will probably want Rust on your computer now. There are always things that you can't do with the Playground like using files or code in more than just one file. Some other things you need Rust on your computer for are input and flags. But most important is that with Rust on your computer you can use crates. We already learned about crates, but in the Playground you could only use the most popular ones. But with Rust on your computer you can use any crate in your program.

