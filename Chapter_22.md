## Tuples
**[See this chapter on YouTube](https://youtu.be/U67Diy6SlTg)**

Tuples in Rust use `()`. We have seen many empty tuples already, because *nothing* in a function actually means an empty tuple:

```text
fn do_something() {}
```

is actually short for:

```text
fn do_something() -> () {}
```

That function gets nothing (an empty tuple), and returns nothing (an empty tuple). So we have been using tuples a lot already. When you don't return anything in a function, you actually return an empty tuple.

```rust
fn just_prints() {
    println!("I am printing"); // Adding ; means we return an empty tuple
}

fn main() {}
```

But tuples can hold many things, and can hold different types too. Items inside a tuple are also indexed with numbers 0, 1, 2, and so on. But to access them, you use a `.` instead of a `[]`. Let's put a whole bunch of types into a single tuple.

```rust
fn main() {
    let random_tuple = ("Here is a name", 8, vec!['a'], 'b', [8, 9, 10], 7.7);
    println!(
        "Inside the tuple is: First item: {:?}
Second item: {:?}
Third item: {:?}
Fourth item: {:?}
Fifth item: {:?}
Sixth item: {:?}",
        random_tuple.0,
        random_tuple.1,
        random_tuple.2,
        random_tuple.3,
        random_tuple.4,
        random_tuple.5,
    )
}
```

This prints:

```text
Inside the tuple is: First item: "Here is a name"
Second item: 8
Third item: ['a']
Fourth item: 'b'
Fifth item: [8, 9, 10]
Sixth item: 7.7
```

That tuple is of type `(&str, i32, Vec<char>, char, [i32; 3], f64)`.


You can use a tuple to create multiple variables. Take a look at this code:

```rust
fn main() {
    let str_vec = vec!["one", "two", "three"];
}
```

`str_vec` has three items in it. What if we want to pull them out? That's where we can use a tuple.

```rust
fn main() {
    let str_vec = vec!["one", "two", "three"];

    let (a, b, c) = (str_vec[0], str_vec[1], str_vec[2]); // call them a, b, and c
    println!("{:?}", b);
}
```

That prints `"two"`, which is what `b` is. This is called *destructuring*. That is because first the variables are inside a structure, but then we made `a`, `b`, and `c` that are not inside a structure.

If you need to destructure but don't want all the variables, you can use `_`.

```rust
fn main() {
    let str_vec = vec!["one", "two", "three"];

    let (_, _, variable) = (str_vec[0], str_vec[1], str_vec[2]);
}
```

Now it only creates a variable called `variable` but doesn't make a variable for the others.

There are many more collection types, and many more ways to use arrays, vecs, and tuples. We will learn more about them too, but first we will learn control flow.

