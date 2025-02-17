## Closures in functions

Closures are great. So how do we put them into our own functions?

You can make your own functions that take closures, but inside them it is less free and you have to decide the type. Outside a function a closure can decide by itself between `Fn`, `FnMut` and `FnOnce`, but inside you have to choose one. The best way to understand is to look at a few function signatures. Here is the one for `.all()`. We remember that it checks an iterator to see if everything is `true` (depending on what you decide is `true` or `false`). Part of its signature says this:


```rust
    fn all<F>(&mut self, f: F) -> bool    // 🚧
    where
        F: FnMut(Self::Item) -> bool,
```

`fn all<F>`: this tells you that there is a generic type `F`. A closure is always generic because every time it is a different type.

`(&mut self, f: F)`: `&mut self` tells you that it's a method. `f: F` is usually what you see for a closure: this is the variable name and the type.  Of course, there is nothing special about `f` and `F` and they could be different names. You could write `my_closure: Closure` if you wanted - it doesn't matter. But in signatures you almost always see `f: F`.

Next is the part about the closure: `F: FnMut(Self::Item) -> bool`. Here it decides that the closure is `FnMut`, so it can change the values. It changes the values of `Self::Item`, which is the iterator that it takes. And it has to return `true` or `false`.

Here is a much simpler signature with a closure:

```rust
fn do_something<F>(f: F)    // 🚧
where
    F: FnOnce(),
{
    f();
}
```

This just says that it takes a closure, takes the value (`FnOnce` = takes the value), and doesn't return anything. So now we can call this closure that takes nothing and do whatever we like. We will create a `Vec` and then iterate over it just to show what we can do now.

```rust
fn do_something<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn main() {
    let some_vec = vec![9, 8, 10];
    do_something(|| {
        some_vec
            .into_iter()
            .for_each(|x| println!("The number is: {}", x));
    })
}
```

For a more real example, we will create a `City` struct again. This time the `City` struct has more data about years and populations. It has a `Vec<u32>` for all the years, and another `Vec<u32>` for all the populations.

`City` has two functions: `new()` to create a new `City`, and `.city_data()` which has a closure. When we use `.city_data()`, it gives us the years and the populations and a closure, so we can do what we want with the data. The closure type is `FnMut` so we can change the data. It looks like this:

```rust
#[derive(Debug)]
struct City {
    name: String,
    years: Vec<u32>,
    populations: Vec<u32>,
}

impl City {
    fn new(name: &str, years: Vec<u32>, populations: Vec<u32>) -> Self {

        Self {
            name: name.to_string(),
            years,
            populations,
        }
    }

    fn city_data<F>(&mut self, mut f: F) // We bring in self, but only f is generic F. f is the closure

    where
        F: FnMut(&mut Vec<u32>, &mut Vec<u32>), // The closure takes mutable vectors of u32
                                                // which are the year and population data
    {
        f(&mut self.years, &mut self.populations) // Finally this is the actual function. It says
                                                  // "use a closure on self.years and self.populations"
                                                  // We can do whatever we want with the closure
    }
}

fn main() {
    let years = vec![
        1372, 1834, 1851, 1881, 1897, 1925, 1959, 1989, 2000, 2005, 2010, 2020,
    ];
    let populations = vec![
        3_250, 15_300, 24_000, 45_900, 58_800, 119_800, 283_071, 478_974, 400_378, 401_694,
        406_703, 437_619,
    ];
    // Now we can create our city
    let mut tallinn = City::new("Tallinn", years, populations);

    // Now we have a .city_data() method that has a closure. We can do anything we want.

    // First let's put the data for 5 years together and print it.
    tallinn.city_data(|city_years, city_populations| { // We can call the input anything we want
        let new_vec = city_years
            .into_iter()
            .zip(city_populations.into_iter()) // Zip the two together
            .take(5)                           // but only take the first 5
            .collect::<Vec<(_, _)>>(); // Tell Rust to decide the type inside the tuple
        println!("{:?}", new_vec);
    });

    // Now let's add some data for the year 2030
    tallinn.city_data(|x, y| { // This time we just call the input x and y
        x.push(2030);
        y.push(500_000);
    });

    // We don't want the 1834 data anymore
    tallinn.city_data(|x, y| {
        let position_option = x.iter().position(|x| *x == 1834);
        if let Some(position) = position_option {
            println!(
                "Going to delete {} at position {:?} now.",
                x[position], position
            ); // Confirm that we delete the right item
            x.remove(position);
            y.remove(position);
        }
    });

    println!(
        "Years left are {:?}\nPopulations left are {:?}",
        tallinn.years, tallinn.populations
    );
}
```

This will print the result of all the times we called `.city_data().` It is:

```text
[(1372, 3250), (1834, 15300), (1851, 24000), (1881, 45900), (1897, 58800)]
Going to delete 1834 at position 1 now.
Years left are [1372, 1851, 1881, 1897, 1925, 1959, 1989, 2000, 2005, 2010, 2020, 2030]
Populations left are [3250, 24000, 45900, 58800, 119800, 283071, 478974, 400378, 401694, 406703, 437619, 500000]
```


