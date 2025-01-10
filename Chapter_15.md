## const and static
**[See this chapter on YouTube](https://youtu.be/Ky3HqkWUcI0)**

There are two other ways to declare values, not just with `let`. These are `const` and `static`. Also, Rust won't use type inference: you need to write the type for them. These are for values that don't change (`const` means constant). The difference is that:

- `const` is for values that don't change, the name is replaced with the value when it's used,
- `static` is similar to `const`, but has a fixed memory location and can act as a global variable.

So they are almost the same. Rust programmers almost always use `const`.

You write them with ALL CAPITAL LETTERS, and usually outside of `main` so that they can live for the whole program.

Two examples are: `const NUMBER_OF_MONTHS: u32 = 12;` and `static SEASONS: [&str; 4] = ["Spring", "Summer", "Fall", "Winter"];`

