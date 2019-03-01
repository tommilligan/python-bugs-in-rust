# Missing Magic Methods

## python: \*-like objects

You may have heard the term _file-like object_ when working in Python. This normally means an object that has `read` and `write` methods.

This means that when we write:

```python
import json

file_handle = open("data.json", "r")
data = json.load(file_handle)
```

We are expected to understand that **implicitly**:

- the builtin `open` function returns an object with a `read` method
- the function `json.load` will call the `read` method on the object it is given
- if either of these constraints is violated, we will get an `AttributeError` at runtime

```python
import json

not_a_file_handle = "spam"
data = json.load(not_a_file_handle) # AttributeError: 'str' object has no attribute 'read'
```

This makes it hard to link external libraries together, all of which may have sligtly different interfaces. Rust has a better solution to explitly enforcing these constraints at compile time: **Traits**.

## rust: Traits

Traits are rust's way of pulling together several concepts found in python:

- magic methods
- abstract base classes
- \*-like objects

In rust we can enforce that a struct has associated methods by defining a trait:

```rust
struct Dog {
    age: i64,
}

trait AnimalAge {
    fn relative_age(&self) -> i64;
}

impl AnimalAge for Dog {
    fn relative_age(&self) -> i64 {
        self.age * 7
    }
}

fn main() {
    let gaspode = Dog {
        age: 9,
    };
    println!("{}", gaspode.relative_age());
}
```

The above example explains that:

- the trait `AnimalAge` contains the method `relative_age`
- `Dog` has an implementation for `AnimalAge`
- `gaspode` is an instance of `Dog`
- therefore, we can call `gaspode.relative_age()` and know the method exists

If we forgot to add our implementation using `impl AnimalAge for Dog`, we would see

```log
error[E0599]: no method named `relative_age` found for type `Dog` in the current scope
  --> /tmp/mdbook-07XN09/display_formatting.md:126:28
   |
2  | struct Dog {
   | ---------- method `relative_age` not found for this
...
14 |     println!("{}", gaspode.relative_age());
   |                            ^^^^^^^^^^^^
   |
   = help: items from traits can only be used if the trait is implemented and in scope
   = note: the following trait defines an item `relative_age`, perhaps you need to implement it:
           candidate #1: `AnimalAge`
```

rustc has helpfully worked out:

- `Dog` does not have a method called `relative_age` directly on it
- there is a trait in scope called `AnimalAge` that we could implement to solve this
