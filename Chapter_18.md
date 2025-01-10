## Giving references to functions
**See this chapter on YouTube: [immutable references](https://youtu.be/mKWXt9YTavc) and [mutable references](https://youtu.be/kJV1wIvAbyk)**

References are very useful for functions. The rule in Rust on values is: a value can only have one owner.

This code will not work:

```rust
fn print_country(country_name: String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Austria");
    print_country(country); // We print "Austria"
    print_country(country); // ⚠️ That was fun, let's do it again!
}
```

It does not work because `country` is destroyed. Here's how:

- Step 1: We create the `String` called `country`. `country` is the owner.
- Step 2: We give `country` to `print_country`. `print_country` doesn't have an `->`, so it doesn't return anything. After `print_country` finishes, our `String` is now dead.
- Step 3: We try to give `country` to `print_country`, but we already did that. We don't have `country` to give anymore.

We can make `print_country` give the `String` back, but it is a bit awkward.

```rust
fn print_country(country_name: String) -> String {
    println!("{}", country_name);
    country_name // return it here
}

fn main() {
    let country = String::from("Austria");
    let country = print_country(country); // we have to use let here now to get the String back
    print_country(country);
}
```

Now it prints:

```text
Austria
Austria
```

The much better way to fix this is by adding `&`.

```rust
fn print_country(country_name: &String) {
    println!("{}", country_name);
}

fn main() {
    let country = String::from("Austria");
    print_country(&country); // We print "Austria"
    print_country(&country); // That was fun, let's do it again!
}
```

Now `print_country()` is a function that takes a reference to a `String`: a `&String`. Also, we give it a reference to country by writing `&country`. This says "you can look at it, but I will keep it".

Now let's do something similar with a mutable reference. Here is an example of a function that uses a mutable variable.

```rust
fn add_hungary(country_name: &mut String) { // first we say that the function takes a mutable reference
    country_name.push_str("-Hungary"); // push_str() adds a &str to a String
    println!("Now it says: {}", country_name);
}

fn main() {
    let mut country = String::from("Austria");
    add_hungary(&mut country); // we also need to give it a mutable reference.
}
```

This prints `Now it says: Austria-Hungary`.

So to conclude:

- `fn function_name(variable: String)` takes a `String` and owns it. If it doesn't return anything, then the variable dies inside the function.
- `fn function_name(variable: &String)` borrows a `String` and can look at it
- `fn function_name(variable: &mut String)` borrows a `String` and can change it

Here is an example that looks like a mutable reference, but it is different.

```rust
fn main() {
    let country = String::from("Austria"); // country is not mutable, but we are going to print Austria-Hungary. How?
    adds_hungary(country);
}

fn adds_hungary(mut country: String) { // Here's how: adds_hungary takes the String and declares it mutable!
    country.push_str("-Hungary");
    println!("{}", country);
}
```

How is this possible? It is because `mut country` is not a reference: `adds_hungary` owns `country` now. (Remember, it takes `String` and not `&String`). The moment you call `adds_hungary`, it becomes the full owner. `country` has nothing to do with `String::from("Austria")` anymore. So `adds_hungary` can take `country` as mutable, and it is perfectly safe to do so.

Remember our employee Powerpoint and manager situation above? In this situation it is like the employee just giving his whole computer to the manager. The employee won't ever touch it again, so the manager can do anything he wants to it.

