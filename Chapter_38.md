## The dbg! macro and .inspect

`dbg!` is a very useful macro that prints quick information. It is a good alternative to `println!` because it is faster to type and gives more information:

```rust
fn main() {
    let my_number = 8;
    dbg!(my_number);
}
```

This prints `[src\main.rs:4] my_number = 8`.

But actually, you can put `dbg!` in many other places, and even wrap code in it. Look at this code for example:

```rust
fn main() {
    let mut my_number = 9;
    my_number += 10;

    let new_vec = vec![8, 9, 10];

    let double_vec = new_vec.iter().map(|x| x * 2).collect::<Vec<i32>>();
}
```

This code creates a new mutable number and changes it. Then it creates a vec, and uses `iter` and `map` and `collect` to create a new vec. We can put `dbg!` almost everywhere in this code. `dbg!` asks the compiler: "What are you doing at this moment?" and tells you.

```rust
fn main() {
    let mut my_number = dbg!(9);
    dbg!(my_number += 10);

    let new_vec = dbg!(vec![8, 9, 10]);

    let double_vec = dbg!(new_vec.iter().map(|x| x * 2).collect::<Vec<i32>>());

    dbg!(double_vec);
}
```

So this prints:

```text
[src\main.rs:3] 9 = 9
```

and:

```text
[src\main.rs:4] my_number += 10 = ()
```

and:

```text
[src\main.rs:6] vec![8, 9, 10] = [
    8,
    9,
    10,
]
```

and this one, which even shows you the value of the expression:

```text
[src\main.rs:8] new_vec.iter().map(|x| x * 2).collect::<Vec<i32>>() = [
    16,
    18,
    20,
]
```

and:

```text
[src\main.rs:10] double_vec = [
    16,
    18,
    20,
]
```

`.inspect` is a bit similar to `dbg!` but you use it like `map` in an iterator. It gives you the iterator item and you can print it or do whatever you want. For example, let's look at our `double_vec` again.

```rust
fn main() {
    let new_vec = vec![8, 9, 10];

    let double_vec = new_vec
        .iter()
        .map(|x| x * 2)
        .collect::<Vec<i32>>();
}
```

We want to know more information about what the code is doing. So we add `inspect()` in two places:

```rust
fn main() {
    let new_vec = vec![8, 9, 10];

    let double_vec = new_vec
        .iter()
        .inspect(|first_item| println!("The item is: {}", first_item))
        .map(|x| x * 2)
        .inspect(|next_item| println!("Then it is: {}", next_item))
        .collect::<Vec<i32>>();
}
```

This prints:

```text
The item is: 8
Then it is: 16
The item is: 9
Then it is: 18
The item is: 10
Then it is: 20
```

And because `.inspect` takes a closure, we can write as much as we want:

```rust
fn main() {
    let new_vec = vec![8, 9, 10];

    let double_vec = new_vec
        .iter()
        .inspect(|first_item| {
            println!("The item is: {}", first_item);
            match **first_item % 2 { // first item is a &&i32 so we use **
                0 => println!("It is even."),
                _ => println!("It is odd."),
            }
            println!("In binary it is {:b}.", first_item);
        })
        .map(|x| x * 2)
        .collect::<Vec<i32>>();
}
```

This prints:

```text
The item is: 8
It is even.
In binary it is 1000.
The item is: 9
It is odd.
In binary it is 1001.
The item is: 10
It is even.
In binary it is 1010.
```

