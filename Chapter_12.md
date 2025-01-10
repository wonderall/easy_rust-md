## The stack, the heap, and pointers

The stack, the heap, and pointers are very important in Rust.

The stack and the heap are two places to keep memory in computers. The important differences are:

- The stack is very fast, but the heap is not so fast. It's not super slow either, but the stack is always faster. But you can't just use the stack all the time, because:
- Rust needs to know the size of a variable at compile time. So simple variables like `i32` go on the stack, because we know their exact size. You always know that an `i32` is going to be 4 bytes, because 32 bits = 4 bytes. So `i32` can always go on the stack.
- But some types don't know the size at compile time. But the stack needs to know the exact size. So what do you do? First you put the data in the heap, because the heap can have any size of data. And then to find it a pointer goes on the stack. This is fine because we always know the size of a pointer. So then the computer first goes to the stack, reads the pointer, and follows it to the heap where the data is.

Pointers sound complicated, but they are easy. Pointers are like a table of contents in a book. Imagine this book:

```text
MY BOOK

TABLE OF CONTENTS

Chapter                        Page
Chapter 1: My life              1
Chapter 2: My cat               15
Chapter 3: My job               23
Chapter 4: My family            30
Chapter 5: Future plans         43
```

So this is like five pointers. You can read them and find the information they are talking about. Where is the chapter "My life"? It's on page 1 (it *points* to page 1). Where is the chapter "My job?" It's on page 23.

The pointer you usually see in Rust is called a **reference**. This is the important part to know: a reference points to the memory of another value. A reference means you *borrow* the value, but you don't own it. It's the same as our book: the table of contents doesn't own the information. It's the chapters that own the information. In Rust, references have a `&` in front of them. So:

- `let my_variable = 8` makes a regular variable, but
- `let my_reference = &my_variable` makes a reference.

You read `my_reference = &my_variable` like this: "my_reference is a reference to my_variable". Or: "my_reference refers to my_variable".

This means that `my_reference` is only looking at the data of `my_variable`. `my_variable` still owns its data.

You can also have a reference to a reference, or any number of references.

```rust
fn main() {
    let my_number = 15; // This is an i32
    let single_reference = &my_number; //  This is a &i32
    let double_reference = &single_reference; // This is a &&i32
    let five_references = &&&&&my_number; // This is a &&&&&i32
}
```

These are all different types, just in the same way that "a friend of a friend" is different from "a friend".

