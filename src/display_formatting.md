# Display Formatting

## python: print

It's common to use `print` statements in python for debugging. For primitive data types, this works well out of the box:

```python
def main():
    wrap = {
        "filling": "falafel"
    }
    print(d)  # {'filling': 'falafel'}
```

However, for more complex types, it can be annoying to run a long task and then get an opaque output:

```python
class Wrap(object):
    def __init__(self, filling):
        self.filling = filling

def main():
    wrap = Wrap("falafel")
    print(wrap)  # <__main__.Wrap object at 0x7f7c24dcd4d0>
```

If what we wanted each time was to see details of our `Wrap`, we could implement the `__repr__` [magic method](https://rszalski.github.io/magicmethods/) which is called by `print`.

```python
class Wrap(object):
    def __init__(self, filling):
        self.filling = filling

    def __repr__(self):
        return f"<Wrap with {self.filling} filling>"

def main():
    wrap = Wrap("falafel")
    print(wrap)  # <Wrap with falafel filling>
```

This has a couple of downsides. Firstly, it can be tedious to reimplement many times. There are a couple of boilerplate libraries out there:

- [`attrs`](https://pypi.org/project/attrs/)
- [`dataclasses` module](https://docs.python.org/3/library/dataclasses.html)

But more importantly, how did we even know to implement `__repr__`? It's not mentioned by `help(print)`, or [the online documentation](https://docs.python.org/3/library/functions.html#print). A quick google for "python print class" does turn up [the right answer](https://stackoverflow.com/questions/1535327/how-to-print-objects-of-class-using-print).

## rust: Derived Traits

In rust, struct method calls are grouped into `Traits`. We can see which trait is needed to display our custom struct from the compiler:

```rust,compile_fail
struct Wrap {
    filling: String
}

fn main() {
    let wrap = Wrap {
        filling: String::from("falafel"),
    };
    println!("{}", wrap)
}
```

```log
error[E0277]: `Wrap` doesn't implement `std::fmt::Display`
  --> /tmp/mdbook-b59VOb/display_formatting.md:61:20
   |
10 |     println!("{}", wrap)
   |                    ^^^^ `Wrap` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Wrap`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
   = note: required by `std::fmt::Display::fmt`
```

This tells us we have a couple of things:

- we cannot format `wrap` in this way with `println!` because it's missing the `fmt::Display` trait
- the function that requires this trait is `fmt::Display::fmt`

We can implement the `Display` trait ourselves:

```rust
use std::fmt;

struct Wrap {
    filling: String
}

impl fmt::Display for Wrap {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "<Wrap with {} filling>", self.filling)
    }
}

fn main() {
    let wrap = Wrap {
        filling: String::from("falafel"),
    };
    println!("{}", wrap) // <Wrap with falafel filling>
}
```

Or we can derive the `Debug` trait that rust provides.

```rust
#[derive(Debug)]
struct Wrap {
    filling: String
}

fn main() {
    let wrap = Wrap {
        filling: String::from("falafel"),
    };
    println!("{:?}", wrap) // Wrap { filling: "falafel" }
}
```

What's going on here? Rust implements the `Debug` trait on all primitive types. We can `derive` this trait for our new struct, if all the fields implement `Debug` themselves.

The `derive` macro will take care of writing the `impl Debug for Wrap` block for use, and asking it to call the corresponding methods on child values (such as `println!("{:?}", self.filling)`).
