## Vectors
**[See this chapter on YouTube](https://youtu.be/Eh-DsRnDKmw)**

In the same way that we have `&str` and `String`, we have arrays and vectors. Arrays are faster with less functionality, and vectors are slower with more functionality. (Of course, Rust is always very fast so vectors are not slow, just slow*er* than arrays.) The type is written `Vec`, and you can also just call it a "vec".

There are two main ways to declare a vector. One is like with `String` using `new`:

```rust
fn main() {
    let name1 = String::from("Windy");
    let name2 = String::from("Gomesy");

    let mut my_vec = Vec::new();
    // If we run the program now, the compiler will give an error.
    // It doesn't know the type of vec.

    my_vec.push(name1); // Now it knows: it's Vec<String>
    my_vec.push(name2);
}
```

You can see that a `Vec` always has something else inside it, and that's what the `<>` (angle brackets) are for. A `Vec<String>` is a vector with one or more `String`s. You can also have more types inside. For example:

- `Vec<(i32, i32)>` this is a `Vec` where each item is a tuple: `(i32, i32)`.
- `Vec<Vec<String>>` this is a `Vec` that has `Vec`s of `Strings`. Say for example you wanted to save your favourite book as a `Vec<String>`. Then you do it again with another book, and get another `Vec<String>`. To hold both books, you would put them into another `Vec` and that would be a `Vec<Vec<String>>`.

Instead of using `.push()` to make Rust decide the type, you can just declare the type.

```rust
fn main() {
    let mut my_vec: Vec<String> = Vec::new(); // The compiler knows the type
                                              // so there is no error.
}
```

You can see that items in vectors must have the same type.

Another easy way to create a vector is with the `vec!` macro. It looks like an array declaration, but has `vec!` in front of it.

```rust
fn main() {
    let mut my_vec = vec![8, 10, 10];
}
```

The type is `Vec<i32>`. You call it a "Vec of i32s". And a `Vec<String>` is a "Vec of strings". And a `Vec<Vec<String>>` is a "Vec of a vec of strings".

You can slice a vector too, just like in an array.

```rust
fn main() {
    let vec_of_ten = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    // Everything is the same as above except we added vec!.
    let three_to_five = &vec_of_ten[2..5];
    let start_at_two = &vec_of_ten[1..];
    let end_at_five = &vec_of_ten[..5];
    let everything = &vec_of_ten[..];

    println!("Three to five: {:?},
start at two: {:?}
end at five: {:?}
everything: {:?}", three_to_five, start_at_two, end_at_five, everything);
}
```

Because a vec is slower than an array, we can use some methods to make it faster. A vec has a **capacity**, which means the space given to the vector. When you push a new item on the vector, it gets closer and closer to the capacity. Then if you go past the capacity, it will make its capacity double and copy the items into the new space. This is called reallocation. We'll use a method called `.capacity()` to look at the capacity of a vector as we add items to it.

For example:

```rust
fn main() {
    let mut num_vec = Vec::new();
    println!("{}", num_vec.capacity()); // 0 elements: prints 0
    num_vec.push('a'); // add one character
    println!("{}", num_vec.capacity()); // 1 element: prints 4. Vecs with 1 item always start with capacity 4
    num_vec.push('a'); // add one more
    num_vec.push('a'); // add one more
    num_vec.push('a'); // add one more
    println!("{}", num_vec.capacity()); // 4 elements: still prints 4.
    num_vec.push('a'); // add one more
    println!("{}", num_vec.capacity()); // prints 8. We have 5 elements, but it doubled 4 to 8 to make space
}
```

This prints:

```text
0
4
4
8
```

So this vector has two reallocations: 0 to 4, and 4 to 8. We can make it faster:

```rust
fn main() {
    let mut num_vec = Vec::with_capacity(8); // Give it capacity 8
    num_vec.push('a'); // add one character
    println!("{}", num_vec.capacity()); // prints 8
    num_vec.push('a'); // add one more
    println!("{}", num_vec.capacity()); // prints 8
    num_vec.push('a'); // add one more
    println!("{}", num_vec.capacity()); // prints 8.
    num_vec.push('a'); // add one more
    num_vec.push('a'); // add one more // Now we have 5 elements
    println!("{}", num_vec.capacity()); // Still 8
}
```

This vector has 0 reallocations, which is better. So if you think you know how many elements you need, you can use `Vec::with_capacity()` to make it faster.

You remember that you can use `.into()` to make a `&str` into a `String`. You can also use it to make an array into a `Vec`. You have to tell `.into()` that you want a `Vec`, but you don't have to choose the type of `Vec`. If you don't want to choose, you can write `Vec<_>`.

```rust
fn main() {
    let my_vec: Vec<u8> = [1, 2, 3].into();
    let my_vec2: Vec<_> = [9, 0, 10].into(); // Vec<_> means "choose the Vec type for me"
                                             // Rust will choose Vec<i32>
}
```

