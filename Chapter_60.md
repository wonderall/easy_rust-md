## A tour of the standard library

Now that you know a lot of Rust, you will be able to understand most things inside the standard library. The code inside it isn't so scary anymore. Let's take a look at some of the parts in it that we haven't learned yet. This tour will go over most parts of the standard library that you don't need to install Rust for. We will revisit a lot of items we already know so we can learn them with greater understanding.

### Arrays

In the past (before Rust 1.53), arrays didn't implement `Iterator` and you needed to use methods like `.iter()` on them in for `loops`. (People also used `&` to get a slice in `for` loops). So this didn't work in the past:

```rust
fn main() {
    let my_cities = ["Beirut", "Tel Aviv", "Nicosia"];

    for city in my_cities {
        println!("{}", city);
    }
}
```

The compiler used to give this message:

```text
error[E0277]: `[&str; 3]` is not an iterator
 --> src\main.rs:5:17
  |
  |                 ^^^^^^^^^ borrow the array with `&` or call `.iter()` on it to iterate over it
```

Luckily, that isn't a problem anymore! So all three of these work:

```rust
fn main() {
    let my_cities = ["Beirut", "Tel Aviv", "Nicosia"];

    for city in my_cities {
        println!("{}", city);
    }
    for city in &my_cities {
        println!("{}", city);
    }
    for city in my_cities.iter() {
        println!("{}", city);
    }
}
```

This prints:

```text
Beirut
Tel Aviv
Nicosia
Beirut
Tel Aviv
Nicosia
Beirut
Tel Aviv
Nicosia
```



If you want to get variables from an array, you can put their names inside `[]` to destructure it. This is the same as using a tuple in `match` statements or destructuring to get variables from a struct.

```rust
fn main() {
    let my_cities = ["Beirut", "Tel Aviv", "Nicosia"];
    let [city1, city2, city3] = my_cities;
    println!("{}", city1);
}
```

This prints `Beirut`.

### char

You can use the `.escape_unicode()` method to get the Unicode number for a `char`:

```rust
fn main() {
    let korean_word = "청춘예찬";
    for character in korean_word.chars() {
        print!("{} ", character.escape_unicode());
    }
}
```

This prints `\u{ccad} \u{cd98} \u{c608} \u{cc2c}`.


You can get a char from `u8` using the `From` trait, but for a `u32` you use `TryFrom` because it might not work. There are many more numbers in `u32` than characters in Unicode. We can see this with a simple demonstration.

```rust
use std::convert::TryFrom; // You need to bring TryFrom in to use it
use rand::prelude::*;      // We will use random numbers too

fn main() {
    let some_character = char::from(99); // This one is easy - no need for TryFrom
    println!("{}", some_character);

    let mut random_generator = rand::thread_rng();
    // This will try 40,000 times to make a char from a u32.
    // The range is 0 (std::u32::MIN) to u32's highest number (std::u32::MAX). If it doesn't work, we will give it '-'.
    for _ in 0..40_000 {
        let bigger_character = char::try_from(random_generator.gen_range(std::u32::MIN..std::u32::MAX)).unwrap_or('-');
        print!("{}", bigger_character)
    }
}
```

Almost every time it will generate a `-`. This is part of the sort of output you will see:

```text
------------------------------------------------------------------------𤒰---------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-------------------------------------------------------------춗--------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
-----------------------------------------------------------------------------------------------
------------򇍜----------------------------------------------------
```

So it's a good thing you need to use `TryFrom`.

Also, as of late August 2020 you can now get a `String` from a `char`. (`String` implements `From<char>`) Just write `String::from()` and put a `char` inside.


### Integers

There are a lot of math methods for these types, plus some others. Here are some of the most useful ones.



`.checked_add()`, `.checked_sub()`, `.checked_mul()`, `.checked_div()`. These are good methods if you think you might get a number that won't fit into a type. They return an `Option` so you can safely check that your math works without making the program panic.

```rust
fn main() {
    let some_number = 200_u8;
    let other_number = 200_u8;

    println!("{:?}", some_number.checked_add(other_number));
    println!("{:?}", some_number.checked_add(1));
}
```

This prints:

```text
None
Some(201)
```


You'll notice that on the page for integers it says `rhs` a lot. This means "right hand side", which is the right hand side when you do some math. For example, in `5 + 6`, `5` is on the left and `6` is on the right, so it's the `rhs`. This is not a keyword, but you will see it a lot so it's good to know.

While we are on the subject, let's learn how to implement `Add`. After you implement `Add`, you can use `+` on a type that you create. You need to implement `Add` yourself because add can mean a lot of things. Here's the example in the standard library page:

```rust
use std::ops::Add; // first bring in Add

#[derive(Debug, Copy, Clone, PartialEq)] // PartialEq is probably the most important part here. You want to be able to compare numbers
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Self; // Remember, this is called an "associated type": a "type that goes together".
                        // In this case it's just another Point

    fn add(self, other: Self) -> Self {
        Self {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}
```

Now let's implement `Add` for our own type. Let's imagine that we want to add two countries together so we can compare their economies. It looks like this:

```rust
use std::fmt;
use std::ops::Add;

#[derive(Clone)]
struct Country {
    name: String,
    population: u32,
    gdp: u32, // This is the size of the economy
}

impl Country {
    fn new(name: &str, population: u32, gdp: u32) -> Self {
        Self {
            name: name.to_string(),
            population,
            gdp,
        }
    }
}

impl Add for Country {
    type Output = Self;

    fn add(self, other: Self) -> Self {
        Self {
            name: format!("{} and {}", self.name, other.name), // We will add the names together,
            population: self.population + other.population, // and the population,
            gdp: self.gdp + other.gdp,   // and the GDP
        }
    }
}

impl fmt::Display for Country {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(
            f,
            "In {} are {} people and a GDP of ${}", // Then we can print them all with just {}
            self.name, self.population, self.gdp
        )
    }
}

fn main() {
    let nauru = Country::new("Nauru", 10_670, 160_000_000);
    let vanuatu = Country::new("Vanuatu", 307_815, 820_000_000);
    let micronesia = Country::new("Micronesia", 104_468, 367_000_000);

    // We could have given Country a &str instead of a String for the name. But we would have to write lifetimes everywhere
    // and that would be too much for a small example. Better to just clone them when we call println!.
    println!("{}", nauru.clone());
    println!("{}", nauru.clone() + vanuatu.clone());
    println!("{}", nauru + vanuatu + micronesia);
}
```

This prints:

```text
In Nauru are 10670 people and a GDP of $160000000
In Nauru and Vanuatu are 318485 people and a GDP of $980000000
In Nauru and Vanuatu and Micronesia are 422953 people and a GDP of $1347000000
```

Later on in this code we could change `.fmt()` to display a number that is easier to read.

The three others are called `Sub`, `Mul`, and `Div`, and they are basically the same to implement. For `+=`, `-=`, `*=` and `/=`, just add `Assign`: `AddAssign`, `SubAssign`, `MulAssign`, and `DivAssign`. You can see the full list [here](https://doc.rust-lang.org/std/ops/index.html#structs), because there are many more. `%` for example is called `Rem`, `-` is called `Neg`, and so on.


### Floats

`f32` and `f64` have a very large number of methods that you use when doing math. We won't look at those, but here are some methods that you might use. They are: `.floor()`, `.ceil()`, `.round()`, and `.trunc()`. All of these return an `f32` or `f64` that is like an integer, with only `0` after the period. They do this:

- `.floor()`: gives you the next lowest integer.
- `.ceil()`: gives you the next highest integer.
- `.round()`: gives you a higher number if 0.5 or more, or the same number if less than 0.5. This is called rounding because it gives you a "round" number (a number that has a short, simple form).
- `.trunc()`: just cuts off the part after the period. Truncate means "to cut off".

Here is a simple function to print them.

```rust
fn four_operations(input: f64) {
    println!(
"For the number {}:
floor: {}
ceiling: {}
rounded: {}
truncated: {}\n",
        input,
        input.floor(),
        input.ceil(),
        input.round(),
        input.trunc()
    );
}

fn main() {
    four_operations(9.1);
    four_operations(100.7);
    four_operations(-1.1);
    four_operations(-19.9);
}
```

This prints:

```text
For the number 9.1:
floor: 9
ceiling: 10
rounded: 9 // because less than 9.5
truncated: 9

For the number 100.7:
floor: 100
ceiling: 101
rounded: 101 // because more than 100.5
truncated: 100

For the number -1.1:
floor: -2
ceiling: -1
rounded: -1
truncated: -1

For the number -19.9:
floor: -20
ceiling: -19
rounded: -20
truncated: -19
```

`f32` and `f64` have a method called `.max()` and `.min()` that gives you the higher or the lower of two numbers. (For other types you can just use `std::cmp::max` and `std::cmp::min`.) Here is a way to use this with `.fold()` to get the highest or lowest number. You can see again that `.fold()` isn't just for adding numbers.

```rust
fn main() {
    let my_vec = vec![8.0_f64, 7.6, 9.4, 10.0, 22.0, 77.345, 10.22, 3.2, -7.77, -10.0];
    let maximum = my_vec.iter().fold(f64::MIN, |current_number, next_number| current_number.max(*next_number)); // Note: start with the lowest possible number for an f64.
    let minimum = my_vec.iter().fold(f64::MAX, |current_number, next_number| current_number.min(*next_number)); // And here start with the highest possible number
    println!("{}, {}", maximum, minimum);
}
```

### bool

In Rust you can turn a `bool` into an integer if you want, because it's safe to do that. But you can't do it the other way around. As you can see, `true` turns to 1 and `false` turns to 0.

```rust
fn main() {
    let true_false = (true, false);
    println!("{} {}", true_false.0 as u8, true_false.1 as i32);
}
```

This prints `1 0`. Or you can use `.into()` if you tell the compiler the type:

```rust
fn main() {
    let true_false: (i128, u16) = (true.into(), false.into());
    println!("{} {}", true_false.0, true_false.1);
}
```

This prints the same thing.

As of Rust 1.50 (released in February 2021), there is now a method called `then()`, which turns a `bool` into an `Option`. With `then()` you write a closure, and the closure is called if the item is `true`. Also, whatever is returned from the closure goes inside the `Option`. Here's a small example:

```rust
fn main() {

    let (tru, fals) = (true.then(|| 8), false.then(|| 8));
    println!("{:?}, {:?}", tru, fals);
}
```

This just prints `Some(8), None`.

And now a bit larger example:

```rust
fn main() {
    let bool_vec = vec![true, false, true, false, false];
    
    let option_vec = bool_vec
        .iter()
        .map(|item| {
            item.then(|| { // Put this inside of map so we can pass it on
                println!("Got a {}!", item);
                "It's true, you know" // This goes inside Some if it's true
                                      // Otherwise it just passes on None
            })
        })
        .collect::<Vec<_>>();

    println!("Now we have: {:?}", option_vec);

    // That printed out the Nones too. Let's filter map them out in a new Vec.
    let filtered_vec = option_vec.into_iter().filter_map(|c| c).collect::<Vec<_>>();

    println!("And without the Nones: {:?}", filtered_vec);
}
```

And here's what this prints:

```text
Got a true!
Got a true!
Now we have: [Some("It\'s true, you know"), None, Some("It\'s true, you know"), None, None]
And without the Nones: ["It\'s true, you know", "It\'s true, you know"]
```

### Vec

Vec has a lot of methods that we haven't looked at yet. Let's start with `.sort()`. `.sort()` is not surprising at all. It uses a `&mut self` to sort a vector.

```rust
fn main() {
    let mut my_vec = vec![100, 90, 80, 0, 0, 0, 0, 0];
    my_vec.sort();
    println!("{:?}", my_vec);
}
```

This prints `[0, 0, 0, 0, 0, 80, 90, 100]`. But there is one more interesting way to sort called `.sort_unstable()`, and it is usually faster. It can be faster because it doesn't care about the order of numbers if they are the same number. In regular `.sort()`, you know that the last `0, 0, 0, 0, 0` will be in the same order after `.sort()`. But `.sort_unstable()` might move the last zero to index 0, then the third last zero to index 2, etc.


`.dedup()` means "de-duplicate". It will remove items that are the same in a vector, but only if they are next to each other. This next code will not just print `"sun", "moon"`:

```rust
fn main() {
    let mut my_vec = vec!["sun", "sun", "moon", "moon", "sun", "moon", "moon"];
    my_vec.dedup();
    println!("{:?}", my_vec);
}
```

It only gets rid of "sun" next to the other "sun", then "moon" next to one "moon", and again with "moon" next to another "moon". The result is: `["sun", "moon", "sun", "moon"]`.

If you want to remove every duplicate, just `.sort()` first:

```rust
fn main() {
    let mut my_vec = vec!["sun", "sun", "moon", "moon", "sun", "moon", "moon"];
    my_vec.sort();
    my_vec.dedup();
    println!("{:?}", my_vec);
}
```

Result: `["moon", "sun"]`.


### String

You will remember that a `String` is kind of like a `Vec`. It is so like a `Vec` that you can do a lot of the same methods. For example, you can start one with `String::with_capacity()`. You want that if you are always going to be pushing a `char` with `.push()` or pushing a `&str` with `.push_str()`. Here's an example of a `String` that has too many allocations.

```rust
fn main() {
    let mut push_string = String::new();
    let mut capacity_counter = 0; // capacity starts at 0
    for _ in 0..100_000 { // Do this 100,000 times
        if push_string.capacity() != capacity_counter { // First check if capacity is different now
            println!("{}", push_string.capacity()); // If it is, print it
            capacity_counter = push_string.capacity(); // then update the counter
        }
        push_string.push_str("I'm getting pushed into the string!"); // and push this in every time
    }
}
```

This prints:

```text
35
70
140
280
560
1120
2240
4480
8960
17920
35840
71680
143360
286720
573440
1146880
2293760
4587520
```

We had to reallocate (copy everything over) 18 times. But now we know the final capacity. So we'll give it the capacity right away, and we don't need to reallocate: just one `String` capacity is enough.

```rust
fn main() {
    let mut push_string = String::with_capacity(4587520); // We know the exact number. Some different big number could work too
    let mut capacity_counter = 0;
    for _ in 0..100_000 {
        if push_string.capacity() != capacity_counter {
            println!("{}", push_string.capacity());
            capacity_counter = push_string.capacity();
        }
        push_string.push_str("I'm getting pushed into the string!");
    }
}
```

And this prints `4587520`. Perfect! We never had to allocate again.

Of course, the actual length is certainly smaller than this. If you try 100,001 times, 101,000 times, etc., it'll still say `4587520`. That's because each time the capacity is two times what it was before. We can shrink it though with `.shrink_to_fit()` (same as for a `Vec`). Our `String` is very large and we don't want to add anything more to it, so we can make it a bit smaller. But only do this if you are sure. Here is why:

```rust
fn main() {
    let mut push_string = String::with_capacity(4587520);
    let mut capacity_counter = 0;
    for _ in 0..100_000 {
        if push_string.capacity() != capacity_counter {
            println!("{}", push_string.capacity());
            capacity_counter = push_string.capacity();
        }
        push_string.push_str("I'm getting pushed into the string!");
    }
    push_string.shrink_to_fit();
    println!("{}", push_string.capacity());
    push_string.push('a');
    println!("{}", push_string.capacity());
    push_string.shrink_to_fit();
    println!("{}", push_string.capacity());
}
```

This prints:

```text
4587520
3500000
7000000
3500001
```

So first we had a size of `4587520`, but we weren't using it all. We used `.shrink_to_fit()` and got the size down to `3500000`. But then we forget that we needed to push an `a` on. When we did that, Rust saw that we needed more space and gave us double: now it's `7000000`. Whoops! So we did `.shrink_to_fit()` again and now it's back down to `3500001`.

`.pop()` works for a `String`, just like for a `Vec`.

```rust
fn main() {
    let mut my_string = String::from(".daer ot drah tib elttil a si gnirts sihT");
    loop {
        let pop_result = my_string.pop();
        match pop_result {
            Some(character) => print!("{}", character),
            None => break,
        }
    }
}
```

This prints `This string is a little bit hard to read.` because it starts from the last character.

`.retain()` is a method that uses a closure, which is rare for `String`. It's just like `.filter()` for an iterator.

```rust
fn main() {
    let mut my_string = String::from("Age: 20 Height: 194 Weight: 80");
    my_string.retain(|character| character.is_alphabetic() || character == ' '); // Keep if a letter or a space
    dbg!(my_string); // Let's use dbg!() for fun this time instead of println!
}
```

This prints:

```text
[src\main.rs:4] my_string = "Age  Height  Weight "
```


### OsString and CString

`std::ffi` is the part of `std` that helps you use Rust with other languages or operating systems. It has types like `OsString` and `CString`, which are like `String` for the operating system or `String` for the language C. They each have their own `&str` type too: `OsStr` and `CStr`. `ffi` means "foreign function interface".

You can use `OsString` when you have to work with an operating system that doesn't have Unicode. All Rust strings are unicode, but not every operating system has it. Here is the simple English explanation from the standard library on why we have `OsString`:

- A string on Unix (Linux, etc.) might be lots of bytes together that don't have zeros. And sometimes you read them as Unicode UTF-8.
- A string on Windows might be made of random 16-bit values that don't have zeros. And sometimes you read them as Unicode UTF-16.
- In Rust, strings are always valid UTF-8, which may contain zeros.

So an `OsString` is made to be read by all of them.

You can do all the regular things with an `OsString` like `OsString::from("Write something here")`. It also has an interesting method called `.into_string()` that tries to make it into a regular `String`. It returns a `Result`, but the `Err` part is just the original `OsString`:

```rust
// 🚧
pub fn into_string(self) -> Result<String, OsString>
```

So if it doesn't work then you just get it back. You can't call `.unwrap()` because it will panic, but you can use `match` to get the `OsString` back. Let's test it out by calling methods that don't exist.

```rust
use std::ffi::OsString;

fn main() {
    // ⚠️
    let os_string = OsString::from("This string works for your OS too.");
    match os_string.into_string() {
        Ok(valid) => valid.thth(),           // Compiler: "What's .thth()??"
        Err(not_valid) => not_valid.occg(),  // Compiler: "What's .occg()??"
    }
}
```

Then the compiler tells us exactly what we want to know:

```text
error[E0599]: no method named `thth` found for struct `std::string::String` in the current scope
 --> src/main.rs:6:28
  |
6 |         Ok(valid) => valid.thth(),
  |                            ^^^^ method not found in `std::string::String`

error[E0599]: no method named `occg` found for struct `std::ffi::OsString` in the current scope
 --> src/main.rs:7:37
  |
7 |         Err(not_valid) => not_valid.occg(),
  |                                     ^^^^ method not found in `std::ffi::OsString`
```

We can see that the type of `valid` is `String` and the type of `not_valid` is `OsString`.

### mem

`std::mem` has some pretty interesting methods. We saw some of them already, such as `.size_of()`, `.size_of_val()` and `.drop()`:


```rust
use std::mem;

fn main() {
    println!("{}", mem::size_of::<i32>());
    let my_array = [8; 50];
    println!("{}", mem::size_of_val(&my_array));
    let mut some_string = String::from("You can drop a String because it's on the heap");
    mem::drop(some_string);
    // some_string.clear();   If we did this it would panic
}
```

This prints:

```text
4
200
```

Here are some other methods in `mem`:

`swap()`: with this you can change the values between two variables. You use a mutable reference for each to do it. This is helpful when you have two things you want to switch and Rust doesn't let you because of borrowing rules. Or just when you want to quickly switch two things.

Here's one example:

```rust
use std::{mem, fmt};

struct Ring { // Create a ring from Lord of the Rings
    owner: String,
    former_owner: String,
    seeker: String, // seeker means "person looking for it"
}

impl Ring {
    fn new(owner: &str, former_owner: &str, seeker: &str) -> Self {
        Self {
            owner: owner.to_string(),
            former_owner: former_owner.to_string(),
            seeker: seeker.to_string(),
        }
    }
}

impl fmt::Display for Ring { // Display to show who has it and who wants it
        fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
            write!(f, "{} has the ring, {} used to have it, and {} wants it", self.owner, self.former_owner, self.seeker)
        }
}

fn main() {
    let mut one_ring = Ring::new("Frodo", "Gollum", "Sauron");
    println!("{}", one_ring);
    mem::swap(&mut one_ring.owner, &mut one_ring.former_owner); // Gollum got the ring back for a second
    println!("{}", one_ring);
}
```

This will print:

```text
Frodo has the ring, Gollum used to have it, and Sauron wants it
Gollum has the ring, Frodo used to have it, and Sauron wants it
```

`replace()`: this is like swap, and actually uses swap inside it, as you can see:

```rust
pub fn replace<T>(dest: &mut T, mut src: T) -> T {
    swap(dest, &mut src);
    src
}
```

So it just does a swap and then returns the other item. With this you replace the value with something else you put in. And since it returns the old value, so you should use it with `let`. Here's a quick example.

```rust
use std::mem;

struct City {
    name: String,
}

impl City {
    fn change_name(&mut self, name: &str) {
        let old_name = mem::replace(&mut self.name, name.to_string());
        println!(
            "The city once called {} is now called {}.",
            old_name, self.name
        );
    }
}

fn main() {
    let mut capital_city = City {
        name: "Constantinople".to_string(),
    };
    capital_city.change_name("Istanbul");
}
```

This prints `The city once called Constantinople is now called Istanbul.`.

One function called `.take()` is like `.replace()` but it leaves the default value in the item. You will remember that default values are usually things like 0, "", and so on. Here is the signature:

```rust
// 🚧
pub fn take<T>(dest: &mut T) -> T
where
    T: Default,
```

So you can do something like this:

```rust
use std::mem;

fn main() {
    let mut number_vec = vec![8, 7, 0, 2, 49, 9999];
    let mut new_vec = vec![];

    number_vec.iter_mut().for_each(|number| {
        let taker = mem::take(number);
        new_vec.push(taker);
    });

    println!("{:?}\n{:?}", number_vec, new_vec);
}
```

And as you can see, it replaced all the numbers with 0: no index was deleted.

```text
[0, 0, 0, 0, 0, 0]
[8, 7, 0, 2, 49, 9999]
```


Of course, for your own type you can implement `Default` to whatever you want. Let's look at an example where we have a `Bank` and a `Robber`. Every time he robs the `Bank`, he gets the money at the desk. But the desk can take money from the back any time, so it always has 50. We will make our own type for this so it will always have 50. Here is how it works:

```rust
use std::mem;
use std::ops::{Deref, DerefMut}; // We will use this to get the power of u32

struct Bank {
    money_inside: u32,
    money_at_desk: DeskMoney, // This is our "smart pointer" type. It has its own default, but it will use u32
}

struct DeskMoney(u32);

impl Default for DeskMoney {
    fn default() -> Self {
        Self(50) // default is always 50, not 0
    }
}

impl Deref for DeskMoney { // With this we can access the u32 using *
    type Target = u32;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

impl DerefMut for DeskMoney { // And with this we can add, subtract, etc.
    fn deref_mut(&mut self) -> &mut Self::Target {
        &mut self.0
    }
}

impl Bank {
    fn check_money(&self) {
        println!(
            "There is ${} in the back and ${} at the desk.\n",
            self.money_inside, *self.money_at_desk // Use * so we can just print the u32
        );
    }
}

struct Robber {
    money_in_pocket: u32,
}

impl Robber {
    fn check_money(&self) {
        println!("The robber has ${} right now.\n", self.money_in_pocket);
    }

    fn rob_bank(&mut self, bank: &mut Bank) {
        let new_money = mem::take(&mut bank.money_at_desk); // Here it takes the money, and leaves 50 because that is the default
        self.money_in_pocket += *new_money; // Use * because we can only add u32. DeskMoney can't add
        bank.money_inside -= *new_money;    // Same here
        println!("She robbed the bank. She now has ${}!\n", self.money_in_pocket);
    }
}

fn main() {
    let mut bank_of_klezkavania = Bank { // Set up our bank
        money_inside: 5000,
        money_at_desk: DeskMoney(50),
    };
    bank_of_klezkavania.check_money();

    let mut robber = Robber { // Set up our robber
        money_in_pocket: 50,
    };
    robber.check_money();

    robber.rob_bank(&mut bank_of_klezkavania); // Rob, then check money
    robber.check_money();
    bank_of_klezkavania.check_money();

    robber.rob_bank(&mut bank_of_klezkavania); // Do it again
    robber.check_money();
    bank_of_klezkavania.check_money();

}
```

This will print:

```text
There is $5000 in the back and $50 at the desk.

The robber has $50 right now.

She robbed the bank. She now has $100!

The robber has $100 right now.

There is $4950 in the back and $50 at the desk.

She robbed the bank. She now has $150!

The robber has $150 right now.

There is $4900 in the back and $50 at the desk.
```

You can see that there is always $50 at the desk.


### prelude

The standard library has a prelude too, which is why you don't have to write things like `use std::vec::Vec` to create a `Vec`. You can see all the items [here](https://doc.rust-lang.org/std/prelude/index.html#prelude-contents), and will already know almost all of them:

- `std::marker::{Copy, Send, Sized, Sync, Unpin}`. You haven't seen `Unpin` before, because it is used for almost every type (like `Sized`, which is also very common). To "pin" means to not let something move. In this case a `Pin` means that it can't move in memory, but most items have `Unpin` so you can. That's why functions like `std::mem::replace` work, because they aren't pinned.
- `std::ops::{Drop, Fn, FnMut, FnOnce}`.
- `std::mem::drop`
- `std::boxed::Box`.
- `std::borrow::ToOwned`. You saw this before a bit with `Cow`, which can take borrowed content and make it owned. It uses `.to_owned()` to do this. You can also use `.to_owned()` on a `&str` to get a `String`, and the same for other borrowed values.
- `std::clone::Clone`
- `std::cmp::{PartialEq, PartialOrd, Eq, Ord}`.
- `std::convert::{AsRef, AsMut, Into, From}`.
- `std::default::Default`.
- `std::iter::{Iterator, Extend, IntoIterator, DoubleEndedIterator, ExactSizeIterator}`. We used `.rev()` for an iterator before: this actually makes a `DoubleEndedIterator`. An `ExactSizeIterator` is just something like `0..10`: it already knows that it has a `.len()` of 10. Other iterators don't know their length for sure.
- `std::option::Option::{self, Some, None}`.
- `std::result::Result::{self, Ok, Err}`.
- `std::string::{String, ToString}`.
- `std::vec::Vec`.

What if you don't want the prelude for some reason? Just add the attribute `#![no_implicit_prelude]`. Let's give it a try and watch the compiler complain:

```rust
// ⚠️
#![no_implicit_prelude]
fn main() {
    let my_vec = vec![8, 9, 10];
    let my_string = String::from("This won't work");
    println!("{:?}, {}", my_vec, my_string);
}
```

Now Rust has no idea what you are trying to do:

```text
error: cannot find macro `println` in this scope
 --> src/main.rs:5:5
  |
5 |     println!("{:?}, {}", my_vec, my_string);
  |     ^^^^^^^

error: cannot find macro `vec` in this scope
 --> src/main.rs:3:18
  |
3 |     let my_vec = vec![8, 9, 10];
  |                  ^^^

error[E0433]: failed to resolve: use of undeclared type or module `String`
 --> src/main.rs:4:21
  |
4 |     let my_string = String::from("This won't work");
  |                     ^^^^^^ use of undeclared type or module `String`

error: aborting due to 3 previous errors
```

So for this simple code you need to tell Rust to use the `extern` (external) crate called `std`, and then the items you want. Here is everything we have to do just to create a Vec and a String and print it:

```rust
#![no_implicit_prelude]

extern crate std; // Now you have to tell Rust that you want to use a crate called std
use std::vec; // We need the vec macro
use std::string::String; // and string
use std::convert::From; // and this to convert from a &str to the String
use std::println; // and this to print

fn main() {
    let my_vec = vec![8, 9, 10];
    let my_string = String::from("This won't work");
    println!("{:?}, {}", my_vec, my_string);
}
```

And now it finally works, printing `[8, 9, 10], This won't work`. So you can see why Rust uses the prelude. But if you want, you don't need to use it. And you can even use `#![no_std]` (we saw this once) for when you can't even use something like stack memory. But most of the time you don't have to think about not using the prelude or `std` at all.

So why didn't we see the `extern` keyword before? It's because you don't need it that much anymore. Before, when bringing in an external crate you had to use it. So to use `rand` in the past, you had to write:

```rust
extern crate rand;
```

and then `use` statements for the mods, traits, etc. that you wanted to use. But the Rust compiler now doesn't need this help anymore - you can just use `use` and it knows where to find it. So you almost never need `extern crate` anymore, but in other people's Rust code you might still see it on the top.



### time

`std::time` is where you can get functions for time. (If you want even more functions, a crate like `chrono` can work.) The simplest function is just getting the system time with `Instant::now()`.

```rust
use std::time::Instant;

fn main() {
    let time = Instant::now();
    println!("{:?}", time);
}
```

If you print it, you'll get something like this: `Instant { tv_sec: 2738771, tv_nsec: 685628140 }`. That's talking about seconds and nanoseconds, but it's not very useful. If you look at 2738771 seconds for example (written in August), it is 31.70 days. That doesn't have anything to do with the month or the day of the year. But the page on `Instant` tells us that it isn't supposed to be useful on its own. It says that it is "opaque and useful only with Duration." Opaque means "you can't figure it out", and duration means "how much time passed". So it's only useful when doing things like comparing times.

If you look at the traits on the left, one of them is `Sub<Instant>`. That means we can use `-` to subtract one from another. And when we click on [src] to see what it does, it says:

```rust
impl Sub<Instant> for Instant {
    type Output = Duration;

    fn sub(self, other: Instant) -> Duration {
        self.duration_since(other)
    }
}
```

So it takes an `Instant` and uses `.duration_since()` to give a `Duration`. Let's try printing that. We'll make two `Instant::now()`s right next to each other, then we'll make the program busy for a while. Then we'll make one more `Instant::now()`. Finally, we'll see how long it took.

```rust
use std::time::Instant;

fn main() {
    let time1 = Instant::now();
    let time2 = Instant::now(); // These two are right next to each other

    let mut new_string = String::new();
    loop {
        new_string.push('წ'); // Make Rust push this Georgian letter onto the String
        if new_string.len() > 100_000 { //  until it is 100,000 bytes long
            break;
        }
    }
    let time3 = Instant::now();
    println!("{:?}", time2 - time1);
    println!("{:?}", time3 - time1);
}
```

This will print something like this:

```text
1.025µs
683.378µs
```

So that's just over 1 microsecond vs. 683 microseconds. We can see that Rust did take some time to do it.

There is one fun thing we can do with just a single `Instant` though. We can turn it into a `String` with `format!("{:?}", Instant::now());`. It looks like this:

```rust
use std::time::Instant;

fn main() {
    let time1 = format!("{:?}", Instant::now());
    println!("{}", time1);
}
```

That prints something like `Instant { tv_sec: 2740773, tv_nsec: 632821036 }`. That's not useful, but if we use `.iter()` and `.rev()` and `.skip(2)`, we can skip the `}` and ` ` at the end. We can use it to make a random number generator.

```rust
use std::time::Instant;

fn bad_random_number(digits: usize) {
    if digits > 9 {
        panic!("Random number can only be up to 9 digits");
    }
    let now = Instant::now();
    let output = format!("{:?}", now);

    output
        .chars()
        .rev()
        .skip(2)
        .take(digits)
        .for_each(|character| print!("{}", character));
    println!();
}

fn main() {
    bad_random_number(1);
    bad_random_number(1);
    bad_random_number(3);
    bad_random_number(3);
}
```

This will print something like:

```text
6
4
967
180
```

The function is called `bad_random_number` because it's not a very good random number generator. Rust has better crates that make random numbers with less code than `rand` like `fastrand`. But it's a good example of how you can use your imagination to do something with `Instant`.

When you have a thread, you can use `std::thread::sleep` to make it stop for a while. When you do this, you have to give it a duration. You don't have to make more than one thread to do this because every program is on at least one thread. `sleep` needs a `Duration` though, so it can know how long to sleep. You can pick the unit like this: `Duration::from_millis()`, `Duration::from_secs`, etc. Here's one example:

```rust
use std::time::Duration;
use std::thread::sleep;

fn main() {
    let three_seconds = Duration::from_secs(3);
    println!("I must sleep now.");
    sleep(three_seconds);
    println!("Did I miss anything?");
}
```

This will just print

```text
I must sleep now.
Did I miss anything?
```

but the thread will do nothing for three seconds. You usually use `.sleep()` when you have many threads that need to try something a lot, like connecting. You don't want the thread to use your processor to try 100,000 times in a second when you just want it to check sometimes. So then you can set a `Duration`, and it will try to do its task every time it wakes up.


### Other macros


Let's take a look at some other macros.

`unreachable!()`

This macro is kind of like `todo!()` except it's for code that you will never do. Maybe you have a `match` in an enum that you know will never choose one of the arms, so the code can never be reached. If that's so, you can write `unreachable!()` so the compiler knows that it can ignore that part.

For example, let's say you have a program that writes something when you choose a place to live in. They are in Ukraine, and all of them are nice except Chernobyl. Your program doesn't let anyone choose Chernobyl, because it's not a good place to live right now. But the enum was made a long time ago in someone else's code, and you can't change it. So in the `match` arm you can use the macro here. It looks like this:

```rust
enum UkrainePlaces {
    Kiev,
    Kharkiv,
    Chernobyl, // Pretend we can't change the enum - Chernobyl will always be here
    Odesa,
    Dnipro,
}

fn choose_city(place: &UkrainePlaces) {
    use UkrainePlaces::*;
    match place {
        Kiev => println!("You will live in Kiev"),
        Kharkiv => println!("You will live in Kharkiv"),
        Chernobyl => unreachable!(),
        Odesa => println!("You will live in Odesa"),
        Dnipro => println!("You will live in Dnipro"),
    }
}

fn main() {
    let user_input = UkrainePlaces::Kiev; // Pretend the user input is made from some other function. The user can't choose Chernobyl, no matter what
    choose_city(&user_input);
}
```

This will print `You will live in Kiev`.

`unreachable!()` is also nice for you to read because it reminds you that some part of the code is unreachable. You have to be sure that the code is actually unreachable though. If the compiler ever calls `unreachable!()`, the program will panic.

Also, if you ever have unreachable code that the compiler knows about, it will tell you. Here is a quick example:

```rust
fn main() {
    let true_or_false = true;

    match true_or_false {
        true => println!("It's true"),
        false => println!("It's false"),
        true => println!("It's true"), // Whoops, we wrote true again
    }
}
```

It will say:

```text
warning: unreachable pattern
 --> src/main.rs:7:9
  |
7 |         true => println!("It's true"),
  |         ^^^^
  |
```

But `unreachable!()` is for when the compiler can't know, like our other example.



`column!`, `line!`, `file!`, `module_path!`

These four macros are kind of like `dbg!()` because you just put them in to give you debug information. But they don't take any variables - you just use them with the brackets and nothing else. They are easy to learn together:

- `column!()` gives you the column where you wrote it,
- `file!()` gives you the name of the file where you wrote it,
- `line!()` gives you the line where you wrote it, and
- `module_path!()` gives you the module where it is.

The next code shows all three in a simple example. We will pretend there is a lot more code (mods inside mods), because that is why we would want to use these macros. You can imagine a big Rust program over many mods and files.

```rust
pub mod something {
    pub mod third_mod {
        pub fn print_a_country(input: &mut Vec<&str>) {
            println!(
                "The last country is {} inside the module {}",
                input.pop().unwrap(),
                module_path!()
            );
        }
    }
}

fn main() {
    use something::third_mod::*;
    let mut country_vec = vec!["Portugal", "Czechia", "Finland"];
    
    // do some stuff
    println!("Hello from file {}", file!());

    // do some stuff
    println!(
        "On line {} we got the country {}",
        line!(),
        country_vec.pop().unwrap()
    );

    // do some more stuff

    println!(
        "The next country is {} on line {} and column {}.",
        country_vec.pop().unwrap(),
        line!(),
        column!(),
    );

    // lots more code

    print_a_country(&mut country_vec);
}
```

It prints this:

```text
Hello from file src/main.rs
On line 23 we got the country Finland
The next country is Czechia on line 32 and column 9.
The last country is Portugal inside the module rust_book::something::third_mod
```



`cfg!`

We know that you can use attributes like `#[cfg(test)]` and `#[cfg(windows)]` to tell the compiler what to do in certain cases. When you have `test`, it will run the code when you run Rust under testing mode (if it's on your computer you type `cargo test`). And when you use `windows`, it will run the code if the user is using Windows. But maybe you just want to change one tiny bit of code depending on the operating system, etc. That's when this macro is useful. It returns a `bool`.

```rust
fn main() {
    let helpful_message = if cfg!(target_os = "windows") { "backslash" } else { "slash" };

    println!(
        "...then in your hard drive, type the directory name followed by a {}. Then you...",
        helpful_message
    );
}
```

This will print differently, depending on your system. The Rust Playground runs on Linux, so it will print:

```text
...then in your hard drive, type the directory name followed by a slash. Then you...
```

`cfg!()` works for any kind of configuration. Here is an example of a function that runs differently when you use it inside a test.

```rust
#[cfg(test)] // cfg! will know to look for the word test
mod testing {
    use super::*;
    #[test]
    fn check_if_five() {
        assert_eq!(bring_number(true), 5); // This bring_number() function should return 5
    }
}

fn bring_number(should_run: bool) -> u32 { // This function takes a bool as to whether it should run
    if cfg!(test) && should_run { // if it should run and has the configuration test, return 5
        5
    } else if should_run { // if it's not a test but it should run, print something. When you run a test it ignores println! statements
        println!("Returning 5. This is not a test");
        5
    } else {
        println!("This shouldn't run, returning 0."); // otherwise return 0
        0
    }
}

fn main() {
    bring_number(true);
    bring_number(false);
}
```

Now it will run differently depending on the configuration. If you just run the program, it will give you this:

```text
Returning 5. This is not a test
This shouldn't run, returning 0.
```

But if you run it in test mode (`cargo test` for Rust on your computer), it will actually run the test. And because the test always returns 5 in this case, it will pass.

```text
running 1 test
test testing::check_if_five ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```



