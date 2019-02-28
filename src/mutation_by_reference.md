# Mutation by reference

## Simple vs complex data structures

Consider this python function:

```python
def main():
    numbers = [0, 1]
    numbers[0] += 100
    print(numbers)
```

We assign two pieces of data in an array, and modify one of them.
As expected, we get:

```python
[100, 1]
```

How about this slightly more complex example?

```python
def main():
    numbers = [0, 1]
    matrix = [numbers, numbers]
    matrix[0].append(2)
    print(matrix)
```

You might expect the output to be:

```python
([0, 1, 2], [0, 1])
```

But what we actually get is:

```python
([0, 1, 2], [0, 1, 2])
```

On line 3, when we assigned an array with `[numbers, numbers]`,
we didn't actually copy the data. We only stored two _references_
to the same piece of data.

So when we update the underlying data, all references are updated.

## References are pointers to data

In rust:
- references are noted explicitly using the `&` operator.
- mutable data structures are noted explicitly using the `mut` keyword.

Here we can see that the `Vec` is mutable, so we should expect the values to change.

```rust
fn main() {
    let mut numbers = vec![0, 1];
    numbers[0] += 100;
    println!("{:?}", numbers); // [100, 1]
}
```

If we write this like our python example, Rust throws us a nice compiler error:

```rust,compile_fail
fn main() {
    let numbers = vec![0, 1];

    // error[E0382]: use of moved value: `numbers`
    let mut matrix = vec![numbers, numbers];

    matrix[0].push(2);
    println!("{:?}", matrix);
}
```

Rust is telling us that we are being ambiguous about how we want to duplicate data here. We can either explicitly use the same `Vec` twice, and store two references to it:

```rust
fn main() {
    let mut numbers = vec![0, 1];

    let mut matrix = [&numbers, &numbers];
    println!("{:?}", matrix); // [[0, 1], [0, 1]]
}
```

Or, we can clone `numbers` to form a second vector, which will allow us to mutate one at a time:

```rust
fn main() {
    let numbers_0 = vec![0, 1];
    let numbers_1 = numbers_0.clone();
    let mut matrix = vec![numbers_0, numbers_1];
    matrix[0].push(2);
    println!("{:?}", matrix); // [[0, 1, 2], [0, 1]]
}
```

Rust makes this difficult as copying a vector is an expensive operation - `Vec` is `Clone`, and not `Copy`.

## Summary

One of rust's core features is _ownership_. This helps avoid
accidental mutation of data, by enforcing who can use and update
data at compile time.
