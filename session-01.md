title: Rust Workshop - Session 1
class: animation-fade
layout: true

.bottom-bar[
{{title}}
]

---

count: false

# Leader slide

- Press 'b' to toggle 'blackout mode'
- Press 'p' to toggle 'presenter mode'
- Press 'c' to create a clone of this window which will keep up
- Press left/right to move around slides
- The next slide is where the presentation begins

???

This is where presenter notes appear

---

class: impact

# {{title}}

## Daniel Silverstone <br /><tt>&lt;dsilvers@digital-scurf.org&gt;</tt>

???

- Welcome everyone
- Explain the intent of the session
    - Namely an introduction

---

title: 
class: middle

# Topics

???

1. Get everyone set up with a toolchain
2. Get everyone comfortable finding docs / introduction to books
3. Example of working environment - VSCode/RLS
4. Some language and syntax fundamentals

---

title: Setting up a toolchain

# Installing Rust in the first place

```shell
$ curl https://sh.rustup.rs/ | sh -
... time passes ...
$ exit
... open new shell ...
$ cargo --version
cargo 1.32.0 (8610973aa 2019-01-02)
```

???

Perform this to install Rust, flip to a terminal to demonstrate.

Use this time to explain about the stable/beta/nightly channels and that
for now at least we'll be using stable.

---

title: Setting up a toolchain

# Adding components

```shell
$ rustup component add clippy rls rustfmt
... time passes ...
$ rustfmt --version
rustfmt 1.0.0-stable (43206f4 2018-11-30)
```

???

* Explain that components are optional parts of the toolchains.
* Explain that the compiler, cargo, rust-std, and rust-docs are mandatory
* Clippy is a linting tool
* RLS is the language service (useful later)
* Rustfmt is for standardising formatting

---

title: IDE integration

* VSCode <https://code.visual-studio.com/>
* `rustup`, `clippy`, `rustfmt` etc
* VSCode's official `rust-lang.rust` extension
    * `Ctrl+P` `ext install rust-lang.rust` `RET`
    * Recommended configurations which differ from default:
    * `editor.formatOnSave` set to `true` for `rustfmt` integration
    * I suggest making clippy `on` but some people prefer `opt-in`

???

1. You will need Visual Studio code, install it now from deb, rpm, or whatever.
2. You have `rustup` etc
3. run `code`
4. Install the extension
5. Make configurations as needed

---

title: Documentation

# Displaying documentation - 'global'

```shell
$ rustup doc --book
...
$ rustup doc --std
...
$ rustup doc --reference
...
```

???

Most of the time we'll be using the std docs, but the book and reference will
prove to be useful background reading if you want to rush ahead.

---

title: Documentation

# Displaying documentation - 'local'

```shell
$ cargo doc --open
...
```

???

Only helpful once you have some code in place, but can provide documentation
for your crate.

---

title: Cargo - getting started

# Creating a crate, running your program

```shell
$ cargo new --bin hello
...
$ cd hello
$ cargo run
Hello world
$
```

???

* Let's all create a hello world application
* Talk about Cargo.toml, src/ dir etc.
* In particular cover the content of Cargo.toml and its implications in terms
  of publishing content.

---

title: Extending Hello World
class: impact

# Extending that…

???

Let's extend this...

* Open src/main.rs in vim
* Write a function to take a str slice to say hello to, pass "world" for now
* This should demonstrate function syntax, also `let` bindings, etc.
* Demonstrate `cargo run`
* use `rustup doc --std` to find `std::env::args()` discuss panics and the
  `_os` variant
* Bring it into scope, and say hello to each argument
* Go back to the docs, show clicking through to `Args` and expand `impl
  Iterator` - Brief chat about iterators

---

title: Development environment
class: impact

# What about VSCode?

???

Sometimes vim on the commandline isn't good enough - show VSCode/RLS

---

title: Ten green bottles
class: impact

# 10 green bottles

???

* Create a new project
* This time we're doing 'ten green bottles'
* To do this, let's start with a function to print a verse
* Encourage them to use u32 or usize for this, since we don't want negative
  bottles.

---

title: Ten green bottles

# Conditional execution

```rust
if condition {
   block;
} else {
   block;
}
```

???

* We want to print a normal verse, but we need to know `bottle` vs `bottles`
* Also when there's no bottles left, we want to say `no green bottles`
* Practice by running calling it with a few different values including
  `10`, and `1`

---

title: Interlude

# Most 'statements' are expressions

`if/else` is an expression:

```rust
let foo = if barcond { something } else { something_else };
```

Leading to:

```rust
fn something() {
    some_expression
}
```

???

Many 'statements' in Rust are actually expressions.  In fact, a statement
is simply an expression which returns unit (`()`).  This leads to functions
often not using the `return` keyword unless they have early returns.

---

title: Interlude

# Other unusual expression forms:

```rust
match something { ... }

loop { ...; break someval; }

```

???

The `match` construct is an expression.  This means that all of the "arms" of
the match must evaluate to the same type, otherwise it can't compile.  If
you're using `match` in statement position, then every arm must be a statement.

Interestingly, the `loop` function, normally a forever loop, can actually
return a value by means of `break`.  Again, any `break`s in a `loop` must have
the same type, so that the loop will have the same type over-all, no matter
the exit path.

---

title: Ten green bottles

# A simple numeric `for` loop

```rust
for round in 1..=10 {
    print_round(round);
}
```

???

Explain the `..=` syntax, ask if anyone can come up with a way to make this
go from 100 down to 1 instead.

Opportunity here to teach them how to work out what the `..=` thing does,
track from there to `Iterator` since we learned that before, and then find
`.rev()` see `DoubleEndedIterator` track back to `RangeInclusive` and see it's
there, and then use it:

The trick is `(1..=10).rev()`

---

title: Ten green bottles

# Parameterising the song

* Get the first argument (default to 10 if no argument provided)
* Display a nice error if the argument is not a useful number

```rust
match maybe_string {
    Some(s) => unimplemented!(),
    None => unimplemented!()
}

some_str.parse().map_err(|_| format!(....))
```

???

* We know that `.next()` on an iterator gives `Option<T>`
* We know that at least the first call of `.next()` on `args()` will be safe
* We can parse a string, look at `str::parse` in the docs
* Parsing returns a Result, upgrade main to return a result
* Now write a function which takes an `Option<String>` from `args()` and
  returns a `Result<u32, String>` or similar.
* Now use match in main to deal with it all.

---

title: Sweetness and light

# A naïve approach

```rust
match something {
    Ok(r) => something_else(r),
    Err(e) => return Err(e),
}
```

???

This kind of match statement is very common, so Rust provides some syntactic
sugar, in the form of an operator, to clean it up...

But first, let's refactor by using `match` as an expression...

---

title: Sweetness and light

# Refactored a bit

```rust
something_else(match something {
    Ok(r) => r,
    Err(e) => return Err(e),
});
```

???

This looks slightly harder to make sense of, but next we can use the sugar...

---

title: Sweetness and light

# Delicious sweet sugary goodness

```rust
something_else(something?)
```

???

The question mark operator let's us clean things up

So let's use it rather than match in our `main` from earlier.

> Switch back to our ten-bottles code and refactor for this.

---

title: Everything is good

* We've installed and set up a development environment
* We have implemented a ten green bottles song program
* This has allowed us to practice using `if` as an expression
* This has taught us about for loops and iterators to some extent
* We have played with some error handling and propagation

???

Does anyone have any questions about what we've covered thus-far?

---

title: Homework

Things for you to research for next week

* These file-associated things:
```rust
std::path::Path
std::path::PathBuf
std::fs::File
std::io::{BufRead, BufReader}
```

* Go and properly grok the following:

```rust
Option<T> / Some(T) / None
Result<T, E> / Ok(T) / Err(E)
```

* Go and research more about iterators, in particular look at some of
  the iterators around the above, such as `PathBuf::ancestors()` or
  `BufRead::lines()`

---

title: Homework
class: impact

# Homework Part 2

???

You will be emailed a couple of specifications for programs which I want each
of you to write ready for the next session.  In terms of code structures,
nothing we've not touched so far today will be needed, though you will need
to grok some more of the standard library as I suggested on the previous slide.

---

count: false
class: impact

# Any questions?

???

Obvious any-questionsness
