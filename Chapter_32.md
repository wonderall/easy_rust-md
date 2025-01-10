## Other collections

Rust has many more types of collections. You can see them at https://doc.rust-lang.org/beta/std/collections/ in the standard library. That page has good explanations for why to use one type, so go there if you don't know what type you want. These collections are all inside `std::collections` in the standard library. The best way to use them is with a `use` statement, like we did with our `enums`. We will start with `HashMap`, which is very common.

### HashMap (and BTreeMap)

A HashMap is a collection made out of *keys* and *values*. You use the key to look up the value that matches the key. You can create a new `HashMap` with just `HashMap::new()` and use `.insert(key, value)` to insert items.

A `HashMap` is not in order, so if you print every key in a `HashMap` together it will probably print differently. We can see this in an example:

```rust
use std::collections::HashMap; // This is so we can just write HashMap instead of std::collections::HashMap every time

struct City {
    name: String,
    population: HashMap<u32, u32>, // This will have the year and the population for the year
}

fn main() {

    let mut tallinn = City {
        name: "Tallinn".to_string(),
        population: HashMap::new(), // So far the HashMap is empty
    };

    tallinn.population.insert(1372, 3_250); // insert three dates
    tallinn.population.insert(1851, 24_000);
    tallinn.population.insert(2020, 437_619);


    for (year, population) in tallinn.population { // The HashMap is HashMap<u32, u32> so it returns a two items each time
        println!("In the year {} the city of {} had a population of {}.", year, tallinn.name, population);
    }
}
```

This prints:

```text
In the year 1372 the city of Tallinn had a population of 3250.
In the year 2020 the city of Tallinn had a population of 437619.
In the year 1851 the city of Tallinn had a population of 24000.
```

or it might print:

```text
In the year 1851 the city of Tallinn had a population of 24000.
In the year 2020 the city of Tallinn had a population of 437619.
In the year 1372 the city of Tallinn had a population of 3250.
```

You can see that it's not in order.

If you want a `HashMap` that you can sort, you can use a `BTreeMap`. Actually they are very similar to each other, so we can quickly change our `HashMap` to a `BTreeMap` to see. You can see that it is almost the same code.

```rust
use std::collections::BTreeMap; // Just change HashMap to BTreeMap

struct City {
    name: String,
    population: BTreeMap<u32, u32>, // Just change HashMap to BTreeMap
}

fn main() {

    let mut tallinn = City {
        name: "Tallinn".to_string(),
        population: BTreeMap::new(), // Just change HashMap to BTreeMap
    };

    tallinn.population.insert(1372, 3_250);
    tallinn.population.insert(1851, 24_000);
    tallinn.population.insert(2020, 437_619);

    for (year, population) in tallinn.population {
        println!("In the year {} the city of {} had a population of {}.", year, tallinn.name, population);
    }
}
```

Now it will always print:

```text
In the year 1372 the city of Tallinn had a population of 3250.
In the year 1851 the city of Tallinn had a population of 24000.
In the year 2020 the city of Tallinn had a population of 437619.
```

Now we will go back to `HashMap`.

You can get a value in a `HashMap` by just putting the key in `[]` square brackets. In this next example we will bring up the value for the key `Bielefeld`, which is `Germany`. But be careful, because the program will crash if there is no key. If you write `println!("{:?}", city_hashmap["Bielefeldd"]);` for example then it will crash, because `Bielefeldd` doesn't exist.

If you are not sure that there will be a key, you can use `.get()` which returns an `Option`. If it exists it will be `Some(value)`, and if not you will get `None` instead of crashing the program. That's why `.get()` is the safer way to get a value from a `HashMap`.

```rust
use std::collections::HashMap;

fn main() {
    let canadian_cities = vec!["Calgary", "Vancouver", "Gimli"];
    let german_cities = vec!["Karlsruhe", "Bad Doberan", "Bielefeld"];

    let mut city_hashmap = HashMap::new();

    for city in canadian_cities {
        city_hashmap.insert(city, "Canada");
    }
    for city in german_cities {
        city_hashmap.insert(city, "Germany");
    }

    println!("{:?}", city_hashmap["Bielefeld"]);
    println!("{:?}", city_hashmap.get("Bielefeld"));
    println!("{:?}", city_hashmap.get("Bielefeldd"));
}
```

This prints:

```text
"Germany"
Some("Germany")
None
```

This is because *Bielefeld* exists, but *Bielefeldd* does not exist.

If a `HashMap` already has a key when you try to put it in, it will overwrite its value:

```rust
use std::collections::HashMap;

fn main() {
    let mut book_hashmap = HashMap::new();

    book_hashmap.insert(1, "L'Allemagne Moderne");
    book_hashmap.insert(1, "Le Petit Prince");
    book_hashmap.insert(1, "섀도우 오브 유어 스마일");
    book_hashmap.insert(1, "Eye of the World");

    println!("{:?}", book_hashmap.get(&1));
}
```

This prints `Some("Eye of the World")`, because it was the last one you used `.insert()` for.

It is easy to check if an entry exists, because you can check with `.get()` which gives an `Option`:

```rust
use std::collections::HashMap;

fn main() {
    let mut book_hashmap = HashMap::new();

    book_hashmap.insert(1, "L'Allemagne Moderne");

    if book_hashmap.get(&1).is_none() { // is_none() returns a bool: true if it's None, false if it's Some
        book_hashmap.insert(1, "Le Petit Prince");
    }

    println!("{:?}", book_hashmap.get(&1));
}
```

This prints `Some("L\'Allemagne Moderne")` because there was already a key for `1`, so we didn't insert `Le Petit Prince`.

`HashMap` has a very interesting method called `.entry()` that you definitely want to try out. With it you can try to make an entry and use another method like `.or_insert()` to insert the value if there is no key. The interesting part is that it also gives a mutable reference so you can change it if you want. First is an example where we just insert `true` every time we insert a book title into the `HashMap`.

Let's pretend that we have a library and want to keep track of our books.

```rust
use std::collections::HashMap;

fn main() {
    let book_collection = vec!["L'Allemagne Moderne", "Le Petit Prince", "Eye of the World", "Eye of the World"]; // Eye of the World appears twice

    let mut book_hashmap = HashMap::new();

    for book in book_collection {
        book_hashmap.entry(book).or_insert(true);
    }
    for (book, true_or_false) in book_hashmap {
        println!("Do we have {}? {}", book, true_or_false);
    }
}
```

This prints:

```text
Do we have Eye of the World? true
Do we have Le Petit Prince? true
Do we have L'Allemagne Moderne? true
```

But that's not exactly what we want. Maybe it would be better to count the number of books so that we know that there are two copies of *Eye of the World*. First let's look at what `.entry()` does, and what `.or_insert()` does. `.entry()` actually returns an `enum` called `Entry`:

```rust
pub fn entry(&mut self, key: K) -> Entry<K, V> // 🚧
```


[Here is the page for Entry](https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html). Here is a simple version of its code. `K` means key and `V` means value.

```rust
// 🚧
use std::collections::hash_map::*;

enum Entry<K, V> {
    Occupied(OccupiedEntry<K, V>),
    Vacant(VacantEntry<K, V>),
}
```

Then when we call `.or_insert()`, it looks at the enum and decides what to do.

```rust
fn or_insert(self, default: V) -> &mut V { // 🚧
    match self {
        Occupied(entry) => entry.into_mut(),
        Vacant(entry) => entry.insert(default),
    }
}
```

The interesting part is that it returns a `mut` reference: `&mut V`. That means you can use `let` to attach it to a variable, and change the variable to change the value in the `HashMap`. So for every book we will insert a 0 if there is no entry. And if there is one, we will use `+= 1` on the reference to increase the number. Now it looks like this:

```rust
use std::collections::HashMap;

fn main() {
    let book_collection = vec!["L'Allemagne Moderne", "Le Petit Prince", "Eye of the World", "Eye of the World"];

    let mut book_hashmap = HashMap::new();

    for book in book_collection {
        let return_value = book_hashmap.entry(book).or_insert(0); // return_value is a mutable reference. If nothing is there, it will be 0
        *return_value +=1; // Now return_value is at least 1. And if there was another book, it will go up by 1
    }

    for (book, number) in book_hashmap {
        println!("{}, {}", book, number);
    }
}
```


The important part is `let return_value = book_hashmap.entry(book).or_insert(0);`. If you take out the `let`, you get `book_hashmap.entry(book).or_insert(0)`. Without `let` it does nothing: it inserts 0, and nobody takes the mutable reference to 0. So we bind it to `return_value` so we can keep the 0. Then we increase the value by 1, which gives at least 1 for every book in the `HashMap`. Then when `.entry()` looks at *Eye of the World* again it doesn't insert anything, but it gives us a mutable 1. Then we increase it to 2, and that's why it prints this:

```text
L'Allemagne Moderne, 1
Le Petit Prince, 1
Eye of the World, 2
```


You can also do things with `.or_insert()` like insert a vec and then push into the vec. Let's pretend that we asked men and women on the street what they think of a politician. They give a rating from 0 to 10. Then we want to put the numbers together to see if the politician is more popular with men or women. It can look like this:


```rust
use std::collections::HashMap;

fn main() {
    let data = vec![ // This is the raw data
        ("male", 9),
        ("female", 5),
        ("male", 0),
        ("female", 6),
        ("female", 5),
        ("male", 10),
    ];

    let mut survey_hash = HashMap::new();

    for item in data { // This gives a tuple of (&str, i32)
        survey_hash.entry(item.0).or_insert(Vec::new()).push(item.1); // This pushes the number into the Vec inside
    }

    for (male_or_female, numbers) in survey_hash {
        println!("{:?}: {:?}", male_or_female, numbers);
    }
}
```

This prints:

```text
"female", [5, 6, 5]
"male", [9, 0, 10]
```

The important line is: `survey_hash.entry(item.0).or_insert(Vec::new()).push(item.1);` So if it sees "female" it will check to see if there is "female" already in the `HashMap`. If not, it will insert a `Vec::new()`, then push the number in. If it sees "female" already in the `HashMap`, it will not insert a new Vec, and will just push the number into it.

### HashSet and BTreeSet

A `HashSet` is actually a `HashMap` that only has keys. On [the page for HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html) it explains this on the top:

`A hash set implemented as a HashMap where the value is ().` So it's a `HashMap` with keys, no values.

You often use a `HashSet` if you just want to know if a key exists, or doesn't exist.

Imagine that you have 100 random numbers, and each number between 1 and 100. If you do this, some numbers will appear more than once, while some won't appear at all. If you put them into a `HashSet` then you will have a list of all the numbers that appeared.

```rust
use std::collections::HashSet;

fn main() {
    let many_numbers = vec![
        94, 42, 59, 64, 32, 22, 38, 5, 59, 49, 15, 89, 74, 29, 14, 68, 82, 80, 56, 41, 36, 81, 66,
        51, 58, 34, 59, 44, 19, 93, 28, 33, 18, 46, 61, 76, 14, 87, 84, 73, 71, 29, 94, 10, 35, 20,
        35, 80, 8, 43, 79, 25, 60, 26, 11, 37, 94, 32, 90, 51, 11, 28, 76, 16, 63, 95, 13, 60, 59,
        96, 95, 55, 92, 28, 3, 17, 91, 36, 20, 24, 0, 86, 82, 58, 93, 68, 54, 80, 56, 22, 67, 82,
        58, 64, 80, 16, 61, 57, 14, 11];

    let mut number_hashset = HashSet::new();

    for number in many_numbers {
        number_hashset.insert(number);
    }

    let hashset_length = number_hashset.len(); // The length tells us how many numbers are in it
    println!("There are {} unique numbers, so we are missing {}.", hashset_length, 100 - hashset_length);

    // Let's see what numbers we are missing
    let mut missing_vec = vec![];
    for number in 0..100 {
        if number_hashset.get(&number).is_none() { // If .get() returns None,
            missing_vec.push(number);
        }
    }

    print!("It does not contain: ");
    for number in missing_vec {
        print!("{} ", number);
    }
}
```

This prints:

```text
There are 66 unique numbers, so we are missing 34.
It does not contain: 1 2 4 6 7 9 12 21 23 27 30 31 39 40 45 47 48 50 52 53 62 65 69 70 72 75 77 78 83 85 88 97 98 99
```

A `BTreeSet` is similar to a `HashSet` in the same way that a `BTreeMap` is similar to a `HashMap`. If we print each item in the `HashSet`, we don't know what the order will be:

```rust
for entry in number_hashset { // 🚧
    print!("{} ", entry);
}
```

Maybe it will print this: `67 28 42 25 95 59 87 11 5 81 64 34 8 15 13 86 10 89 63 93 49 41 46 57 60 29 17 22 74 43 32 38 36 76 71 18 14 84 61 16 35 90 56 54 91 19 94 44 3 0 68 80 51 92 24 20 82 26 58 33 55 96 37 66 79 73`. But it will almost never print it in the same way again.

Here as well, it is easy to change your `HashSet` to a `BTreeSet` if you decide you need ordering. In our code, we only need to make two changes to switch from a `HashSet` to a `BTreeSet`.

```rust
use std::collections::BTreeSet; // Change HashSet to BTreeSet

fn main() {
    let many_numbers = vec![
        94, 42, 59, 64, 32, 22, 38, 5, 59, 49, 15, 89, 74, 29, 14, 68, 82, 80, 56, 41, 36, 81, 66,
        51, 58, 34, 59, 44, 19, 93, 28, 33, 18, 46, 61, 76, 14, 87, 84, 73, 71, 29, 94, 10, 35, 20,
        35, 80, 8, 43, 79, 25, 60, 26, 11, 37, 94, 32, 90, 51, 11, 28, 76, 16, 63, 95, 13, 60, 59,
        96, 95, 55, 92, 28, 3, 17, 91, 36, 20, 24, 0, 86, 82, 58, 93, 68, 54, 80, 56, 22, 67, 82,
        58, 64, 80, 16, 61, 57, 14, 11];

    let mut number_btreeset = BTreeSet::new(); // Change HashSet to BTreeSet

    for number in many_numbers {
        number_btreeset.insert(number);
    }
    for entry in number_btreeset {
        print!("{} ", entry);
    }
}
```

Now it will print in order: `0 3 5 8 10 11 13 14 15 16 17 18 19 20 22 24 25 26 28 29 32 33 34 35 36 37 38 41 42 43 44 46 49 51 54 55 56 57 58 59 60 61 63 64 66 67 68 71 73 74 76 79 80 81 82 84 86 87 89 90 91 92 93 94 95 96`.

### BinaryHeap

A `BinaryHeap` is an interesting collection type, because it is mostly unordered but has a bit of order. It keeps the largest item in the front, but the other items are in any order.

We will use another list of items for an example, but this time smaller.

```rust
use std::collections::BinaryHeap;

fn show_remainder(input: &BinaryHeap<i32>) -> Vec<i32> { // This function shows the remainder in the BinaryHeap. Actually an iterator would be
                                                         // faster than a function - we will learn them later.
    let mut remainder_vec = vec![];
    for number in input {
        remainder_vec.push(*number)
    }
    remainder_vec
}

fn main() {
    let many_numbers = vec![0, 5, 10, 15, 20, 25, 30]; // These numbers are in order

    let mut my_heap = BinaryHeap::new();

    for number in many_numbers {
        my_heap.push(number);
    }

    while let Some(number) = my_heap.pop() { // .pop() returns Some(number) if a number is there, None if not. It pops from the front
        println!("Popped off {}. Remaining numbers are: {:?}", number, show_remainder(&my_heap));
    }
}
```

This prints:

```text
Popped off 30. Remaining numbers are: [25, 15, 20, 0, 10, 5]
Popped off 25. Remaining numbers are: [20, 15, 5, 0, 10]
Popped off 20. Remaining numbers are: [15, 10, 5, 0]
Popped off 15. Remaining numbers are: [10, 0, 5]
Popped off 10. Remaining numbers are: [5, 0]
Popped off 5. Remaining numbers are: [0]
Popped off 0. Remaining numbers are: []
```

You can see that the number in the 0 index is always largest: 25, 20, 15, 10, 5, then 0. But the other ones are all different.

A good way to use a `BinaryHeap` is for a collection of things to do. Here we create a `BinaryHeap<(u8, &str)>` where the `u8` is a number for the importance of the task. The `&str` is a description of what to do.

```rust
use std::collections::BinaryHeap;

fn main() {
    let mut jobs = BinaryHeap::new();

    // Add jobs to do throughout the day
    jobs.push((100, "Write back to email from the CEO"));
    jobs.push((80, "Finish the report today"));
    jobs.push((5, "Watch some YouTube"));
    jobs.push((70, "Tell your team members thanks for always working hard"));
    jobs.push((30, "Plan who to hire next for the team"));

    while let Some(job) = jobs.pop() {
        println!("You need to: {}", job.1);
    }
}
```

This will always print:

```text
You need to: Write back to email from the CEO
You need to: Finish the report today
You need to: Tell your team members thanks for always working hard
You need to: Plan who to hire next for the team
You need to: Watch some YouTube
```

### VecDeque

A `VecDeque` is a `Vec` that is good at popping items both off the front and the back. Rust has `VecDeque` because a `Vec` is great for popping off the back (the last item), but not so great off the front. When you use `.pop()` on a `Vec`, it just takes off the last item on the right and nothing else is moved. But if you take it off another part, all the items to the right are moved over one position to the left. You can see this in the description for `.remove()`:


```text
Removes and returns the element at position index within the vector, shifting all elements after it to the left.
```

So if you do this:

```rust
fn main() {
    let mut my_vec = vec![9, 8, 7, 6, 5];
    my_vec.remove(0);
}
```

it will remove `9`. `8` in index 1 will move to index 0, `7` in index 2 will move to index 1, and so on. Imagine a big parking lot where every time one car leaves all the cars on the right side have to move over.

This, for example, is a *lot* of work for the computer. In fact, if you run it on the Playground it will probably just give up because it's too much work.

```rust
fn main() {
    let mut my_vec = vec![0; 600_000];
    for i in 0..600000 {
        my_vec.remove(0);
    }
}
```

This is a `Vec` of 600,000 zeros. Every time you use `remove(0)` on it, it moves each zero left one space to the left. And then it does it 600,000 times.

You don't have to worry about that with a `VecDeque`. It is usually a bit slower than a `Vec`, but if you have to do things on both ends then it is much faster. You can just use `VecDeque::from` with a `Vec` to make one. Our code above then looks like this:

```rust
use std::collections::VecDeque;

fn main() {
    let mut my_vec = VecDeque::from(vec![0; 600000]);
    for i in 0..600000 {
        my_vec.pop_front(); // pop_front is like .pop but for the front
    }
}
```

It is now much faster, and on the Playground it finishes in under a second instead of giving up.

In this next example we have a `Vec` of things to do. Then we make a `VecDeque` and use `.push_front()` to put them at the front, so the first item we added will be on the right. But each item we push is a `(&str, bool)`: `&str` is the description and `false` means it's not done yet. We use our `done()` function to pop an item off the back, but we don't want to delete it. Instead, we change `false` to `true` and push it at the front so that we can keep it.

It looks like this:

```rust
use std::collections::VecDeque;

fn check_remaining(input: &VecDeque<(&str, bool)>) { // Each item is a (&str, bool)
    for item in input {
        if item.1 == false {
            println!("You must: {}", item.0);
        }
    }
}

fn done(input: &mut VecDeque<(&str, bool)>) {
    let mut task_done = input.pop_back().unwrap(); // pop off the back
    task_done.1 = true;                            // now it's done - mark as true
    input.push_front(task_done);                   // put it at the front now
}

fn main() {
    let mut my_vecdeque = VecDeque::new();
    let things_to_do = vec!["send email to customer", "add new product to list", "phone Loki back"];

    for thing in things_to_do {
        my_vecdeque.push_front((thing, false));
    }

    done(&mut my_vecdeque);
    done(&mut my_vecdeque);

    check_remaining(&my_vecdeque);

    for task in my_vecdeque {
        print!("{:?} ", task);
    }
}
```

This prints:

```text
You must: phone Loki back
("add new product to list", true) ("send email to customer", true) ("phone Loki back", false)
```

