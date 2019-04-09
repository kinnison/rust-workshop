title: Rust Workshop - Session 5
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
- Explain the purpose of the session
  - Some more trait related stuff, including advanced bits
  - Learn about lifetimes and how borrows use them

---

title:
class: middle

# Topics

1. Going over homework
2. Defining your own traits
3. Another way to say what type you take as an argument
4. Lifetimes - What are they and why do they matter
5. Specifying lifetimes when defining types

---

title:
class: impact

# Handing in Homework

???

Let's go over your homework questions and ways in which you might have
solved them.

---

title: Defining traits

```rust
pub trait UsefulThing {
    fn must_be_provided(&self) -> usize;

    fn can_be_overridden(&self) -> Option<usize> {
        None
    }
}
```

???

* Defining traits of your own is super-useful and in a lot of case necessary.
* You can define methods which must be provided by implementations of the trait
  such as the `next()` method of `Iterator`
* You can define methods which provide default implementations such as `map()`
  in `Iterator`
* Default implementations can only rely on the trait methods.
* But...

---

title: Advanced traits - Supertraits

```rust
pub trait MyThing : Clone {
    fn make_fresh_copy(&self) -> Self { self.clone() }
}
```

???

* If your trait requires other traits to be implemented too, then you can rely
  on those in addition.
* If you use the `: AnotherTrait` syntax you can specify dependencies for your
  trait.  A type which implements `MyThing` **must** also implement `Clone`
* This is a little bit like inheritance in an OO model

---

title: Advanced traits - associated types

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Self::Item;
}
```

???

* Associated types are types which are associated with a trait but which are
  not specified by the trait directly.
* We've seen associated types before in `Iterator`
* An associated type is *NOT* a type parameter
* This means there can only be one `Iterator` implementation for a concrete
  type.
* It also means you don't have to keep annotating the type parameter in
  everything.

---

title: `impl Trait`

## `impl Trait` in return position

```rust
fn something() -> impl SomeTrait { ... }
```

???

* Allows the caller to know only that the given trait is implemented by the
  return type
* Still only allows the function to return a single concrete type (no inherent
  dynamic dispatch here)
* Permits the return of something whose type is super-hard to name, such as
  closures which implement `Fn`, `FnOnce`, or `FnMut`, but whose type cannot
  be written out.
* The type is inferred from the function body rather than being passed in from
  the outside and constraining the function.

---

title: `impl Trait`

## `impl Trait` in parameter position

```rust
fn something(reader: &mut impl Read) -> io::Result<()> { ... }
```

???

* Slightly less verbose than the more traditional `where` approach or clearer
  than the angle brackets with types `<R: Read>`, especially where multiple
  arguments with the same constraints (but not needing the same type) must be
  provided.
* Still monomorphises

---

title: `dyn Trait`

```rust
// What is OneThing
// What is AnotherThing
// Are they structs/enums, or are they trait names?

Box<OneThing>
Box<AnotherThing>
```

???

* Sometimes it could be hard to tell if a type was concrete (struct/enum) or
  a trait.
* The `dyn Trait` syntax means this is clearer.

---

title: `dyn Trait`

```rust
// A struct, thus the box is a ptr
Box<OneThing>

// A trait, thus the box is two ptrs (value and trait vtable)
Box<dyn AnotherThing>
```

???

* This helps humans - the compiler could always tell which was which, but
  humans couldn't
* It's better to add syntax so that different things look different

---

title: `dyn Trait`

```rust
fn borrow_thing(a: &Thing) { ... }

// vs.

fn borrow_thing(a: &dyn Thing) { ... }
```

???

* Even borrows can use `dyn` (and in fact `impl`) to make it clear that
  `Thing` is a trait, not a struct.
* The first example will be monomorphized - one version of the function for
  each argument type used in your program.
* The second example will use dynamic dispatch - one version shared among every
  argument type.
* The former can be faster
* The latter can be smaller
* Tradeoffs should be made based on how you see your code being used.

---

title: `impl Trait` and `dyn Trait`

# Useful to know...

* The compiler can lint this for you…

???

The compiler can nudge you toward better code, though for now don't worry
too much about writing it, just be aware when reading others' code.

---

title: `impl Trait` and `dyn Trait`

# Useful to know...

* The compiler can lint this for you…
* If you add `#![deny(bare_trait_objects)]` to your code.

???

* You just have to ask it to.
* It's possible (and likely) that this will become the default in some future
  version of the compiler, and clippy may already be ranting at you about it.

---

title: Lifetimes
class: impact

# Lifetimes

???

* Lifetimes are part of the type system in Rust
* Most of the time, the compiler can work out all the lifetimes for itself
* Sometimes it requires that you join in, either to disambiguate, or else
  to prove you understand them too

---

title: Lifetimes - elided

```rust
fn get_first(input: &[String]) -> Option<&String> {...}
```

???

* Can anyone see where the lifetimes in this example might live?

---

title: Lifetimes - elided - reintroduced

```rust
fn get_first<'a>(input: &'a [String]) -> Option<&'a String> {...}
```

???

* Lifetime names start with an apostrophe or 'tick'.
* The name `'a` is arbitrary.
* In the absence of any information from you, the compiler does its best by
  simply assuming all incoming borrows have independent lifetimes and that
  outgoing borrows come from them in some way if there's only one.
* This means that the compiler is saying that the returned String borrow
  inside the option will live only as long as whatever the input slice was
  borrowed from.
* If there is more than one incoming borrow, the compiler cannot infer the
  lifetime of any outgoing borrows and so will complain and ask you to tell it.

---

title: Lifetimes - structs

```rust
struct MyThing {
    other: &SomethingElse,
}
```

???

* Sometimes you don't want your structs to own stuff, but rather to borrow it
* But what happens if you construct a `MyThing` and then the `SomethingElse`
  it is borrowing goes out of scope.
* That would perhaps induce a use-after-free type bug, which is something that
  the Rust compiler tries to avoid.

---

title: Lifetimes - structs - be explicit

```rust
struct MyThing<'a> {
    other: &'a SomethingElse,
}
```

???

* Again, the `'a` name is arbitrary
* This time, we tell the compiler that the `MyThing` struct type has a lifetime
  parameter which is applied to the `other` borrow of the `SomethingElse`.
* This means that the compiler is able to track the borrow and thus prevent us
  from use-after-free and other bugs.
* Fortunately we don't tend to have to specify the lifetime argument when using
  the type, because now that we've taught the compiler that we're aware of how
  the lifetimes relate, it can track them for us instead.

---

title: Lifetimes - structs - implementation

```rust
struct MyThing<'a> {
    other: &'a SomethingElse,
}

impl<'a> MyThing<'a> {
    ...
}
```

???

* When writing `impl` blocks for a struct which has parameters you have to
  specify the parameters in the `impl` block too.
* This extends to lifetime parameters, despite what I just said.  You **must**
  be explicit here.
* The `'a` in the `impl` does not have to match the `'a` in the `struct`
  definition, it's merely convention that it does.

---

title: Lifetimes - structs - implementation - elision

```rust
impl<'a> MyThing<'a> {
    fn some_method(&mut self) {...}
}
```

???

* Fortunately due to the lifetime elision rules, we don't have to then add
  the `'a` to every borrow of `self`.
* In the future, the compiler may learn even more elision rules, making life
  easier and reducing the number of lifetime annotations that the compiler
  demands you write.  But it will always remain valid to be explicit about
  them.

---

title: Lifetimes - Special lifetimes

```rust
const MESSAGE: &'static str = "Hello World";
```

???

* There is a special lifetime name `'static` which tells the compiler that the
  borrow will remain valid for the duration of the execution of the program.
* You will encounter `'static` in error message suggestions at times, as your
  programs become more complex.
* However, don't immediately jump to `'static`, instead think about why the
  compiler is suggesting you'd need the borrow to last the lifetime of the
  program and decide if there's perhaps some way you can improve matters either
  by transferring ownership, cloning/copying, or fixing some other assumption
  about lifetimes in your code.

---

title: Homework

If you didn't last week, then research `std::io::Cursor` because we'll be using
it in today's homework.

Read, re-read, play with, understand, the "Traits: Defining Shared Behaviour"
chapter of the rust book (10.2 from `rustup doc --book`)

At least skim, but ideally internalise, the "Advanced Traits" chapter of the
rust book (19.3 from `rustup doc --book`)-

Read, re-read, play with, understand, really *really* grok, the "Validating
References with lifetimes" chapter of the rust book (10.3 from `rustup doc
--book`)

- The tasks will be emailed to you like last time.

---

count: false
class: impact

# Any questions?

???

Obvious any-questionsness
