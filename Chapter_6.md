## Comments
**[See this chapter on YouTube](https://youtu.be/fJ7jBZG_Rpo)**

Comments are made for programmers to read, not the computer. It's good to write comments to help other people understand your code.  It's also good to help you understand your code later.  (Many people write good code but then forget why they wrote it.) To write comments in Rust you usually use `//`:

```rust
fn main() {
    // Rust programs start with fn main()
    // You put the code inside a block. It starts with { and ends with }
    let some_number = 100; // We can write as much as we want here and the compiler won't look at it
}
```

When you do this, the compiler won't look at anything to the right of the `//`.

There is another kind of comment that you write with `/*` to start and `*/` to end. This one is useful to write in the middle of your code.

```rust
fn main() {
    let some_number/*: i16*/ = 100;
}
```

To the compiler, `let some_number/*: i16*/ = 100;` looks like `let some_number = 100;`.

The `/* */` form is also useful for very long comments over more than one line. In this example you can see that you need to write `//` for every line. But if you type `/*`, it won't stop until you finish it with `*/`.

```rust
fn main() {
    let some_number = 100; /* Let me tell you
    a little about this number.
    It's 100, which is my favourite number.
    It's called some_number but actually I think that... */

    let some_number = 100; // Let me tell you
    // a little about this number.
    // It's 100, which is my favourite number.
    // It's called some_number but actually I think that...
}
```

