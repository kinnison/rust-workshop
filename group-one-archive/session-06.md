title: Rust Workshop - Session 6
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
  - Marker traits
  - Some standard library features worth reiterating
  - Ways to test that things fail

---

title:
class: middle

# Topics

1. Going over homework
2. Marker traits and the like
3. Some `Option` and `Result` features worth remembering
4. Testing things that should fail

---

title:
class: impact

# Handing in Homework

???

Let's go over your homework questions and ways in which you might have
solved them.

---

title: Marker traits
class: impact

## What are "marker" traits?

???

Marker traits are traits which the compiler can automatically infer
implementations of for your data types, they are intrinsic to the core data
types in some fashion, and are usually entirely automatic.

Marker traits have no functions or associated types and are basically just
type markers for the compiler to infer correctness about your code.

---

title: Marker traits

# Examples

1. `Sized`

???

The `Sized` marker trait simply means that, at compile time, the compiler can
know exactly how many bytes the type will consume.  For example, `u32` is
`Sized` but `[u32]` is not.

---

title: Marker traits

1. `Sized`
2. `Send` and `Sync`

???

`Send` means that the value is safe to be transferred to another thread.  i.e.
it contains nothing which can't traverse a thread boundary safely.

`Sync` means that the value is safe to be referentially *shared* between
threads.  i.e. that it's not a problem if more than one thread accesses the
type at the same time (in a non `mut` sense)

---

title: Marker traits

1. `Sized`
2. `Send` and `Sync`
3. `Copy`

???

`Copy` means that the compiler is good to copy the literal bits that represent
the value and that won't introduce badness.  For example, a `u32` is safe to
`Copy` but if a `u32` were holding a FD which might get closed on `Drop` then
it would not be `Copy` because that'd mean you could end up closing the FD more
than once which isn't good.

Typically you derive `Copy` on any struct or enum you create.

`Clone` is a super-trait of `Copy` and if you derive `Copy` you almost must
derive `Clone`.  You can think of `Copy` as a shallow copy and `Clone` as a
deep copy.

---

title: Marker traits

1. `Sized`
2. `Send` and `Sync`
3. `Copy`
4. `Unpin`

???

`Unpin` is new and interesting and requires a lot more time dedicated to it
than I want for this session.  Suffice it to say that it's a way to tell the
compiler that it's okay for the value to move around in memory after the
compiler has decided that it really shouldn't move if possible.  Mostly you'll
not need to know about this, but you may start to encounter `Pin` and `Unpin`
in some APIs so it's worth being aware of even if you are not knowledgeable
about it.

---

title: Not quite marker traits

1. What?

???

By a "not quite marker trait" I mean a trait which doesn't *require* you to
implement any particular methods, but is also more featured than a true marker
trait.

---

title: Not quite marker traits

1. What?
2. `Error` -> `impl Error for MyType {}`

???

`Error` is one such trait.  It has default implementations for all its methods,
though you can replace them selectively if you so choose.  But for those
methods to be provided for you, your type must implement `Debug` and `Display`
which are supertraits of `Error`.

---

title: Not quite marker traits

1. What?
2. `Error` -> `impl Error for MyType {}`
3. `Eq` -> `impl Eq for MyType {}`

???

`Eq` is an interesting one, it really does have no methods, no associated items
of any kind.  It requires that the type implements `PartialEq` and merely
tightens the equality rules from simple a/b comparisons to full-on reflexivity
(`a==a`) symmetry (`a==b` means `b==a`) and transitivity (`a==b` and `b==c`
means `a==c`).

---

title: Not quite marker traits

1. What?
2. `Error` -> `impl Error for MyType {}`
3. `Eq` -> `impl Eq for MyType {}`
4. `FusedIterator` -> `impl FusedIterator for MyType {}`

???

Along the same lines, `FusedIterator` marks any `Iterator` for which the
`next()` method will safely return `None` again and again once the iteration
is over.  This is not required by `Iterator` (calling `next()` after the first
`None` on an `Iterator` leads to undefined behaviour), but if an iterator is
safe to keep calling, by marking it as fused, certain things become a little
more efficient.

There are more of these, but it's enough to know about these few and more
importantly, how to implement them if you can't derive them automatically.

---

title: `Option`s as `Result`s and vice-versa

* `Option::ok_or()`
* `Option::ok_or_else()`
* `Option::transpose()`
* `Result::ok()`
* `Result::err()`
* `Result::transpose()`

???

There is a close relationship between `Option`s and `Result`s.  In particular,
one can think if an `Option` as a `Result` whose error type is never used,
and a `Result` as an `Option` of the `Ok` type, and an `Option` of the `Err`
type, of which only one can be `Some` at a time.

Given that, you can convert between the two pretty easily using these
functions.  The `transpose` ones are the most fun -- they convert between
`Option<Result<T,E>>` and `Result<Option<T>,E>`.  Really handy when you need
to pop the option/result relationship around.

---

title: `Option`s and `Result`s are iterators

* `Option::iter()`
* `Result::iter()`

???

Both of this iterators will either yield one or zero items before stopping.

It's super-helpful to know that option and result are iterators because it
means you can use the `Iterator::flat_map()` function to flatten things out.
We've seen this before when parsing BF programs.

---

title: Borrowing things inside `Option`s and `Results`

* `as_ref()`
* `as_mut()`

???

If you have an owned option, but you perhaps want to do something with a borrow
of what's inside it, you can use the `as_ref()` and `as_mut()` functions to
get a borrow, or a mutable borrow, of the thing inside the option or result.

Basically this means that, for example, `unwrap()` won't consume the actual
owned option or result, but rather consume a referrential one, which is
handy when you've determined the unwrap would be safe but you can't move
the value.

---

title: Taking and replacing `Option` values

* `Option::take()`
* `Option::replace()`

???

These functions both let you modify an option.  In the former case, you take
any stored value out of the option leaving `None` behind.  In the latter, you
leave `Some(T)` behind instead.  Both return the original value (either `None`
or `Some(T)`).

This is a super-useful pattern for doing partially initialised structures and
also for gently deinitialising bit during a `Drop` implementation.

---

title: Testing things that should fail - `Option` and `Result`

```rust
#[test]
fn test_this_should_fail() {
   assert!(this_should_return_none().is_none());
   assert!(this_should_be_err().is_err());
   assert_eq!(this_should_err_five().err().expect("Wasn't an err?"), 5);
}
```

???

Pretty easy to do for `Option` and `Result` types, you can assert the
`None`ness or `Err`ness of the value.  In the `Err` case, you could even
extract it and assert the value of the error if you so choose:

---

title: Testing things that should fail - `panic!()`

```rust
#[test]
#[should_panic]
fn this_goes_boom() {
    panic!("boom");
}
```

???

The `should_panic` attribute on a test tells the test runner that this should
fail if the test *does not* panic.  However this can be imprecise because
all we get on a success is that *some* panic occurred, and on failure we get
no indication of *what* panic we were hoping for

---

title: Testing things that should fail - `panic!()`

```rust
#[test]
#[should_panic(expected = "boom")]
fn this_goes_boom() {
    panic!("boom bangalang");
}
```

???

This time, the test will fail because while it `panic!()`d, it wasn't the
right kind of panic (wrong message).

---

title: Testing things with results

```rust
#[test]
fn something_testy() -> Result<T, E> {
    ...
}
```

???

Yes it's possible to test things by returning a `Result` object too.  However
something failing here will fail the test, and a success value wouldn't be
useful so we wouldn't want to transpose `<T, E>` to `<E, T>`, so this is
less helpful when we're trying to test failures.

---

title: Homework

* Look at the `Drop` trait
* Remind yourself about the `Read` and `Write` traits
* Have a peek at `std::io::stdout` and `std::io::stdin`
* Properly appreciate how `Display` works when you can't derive it

Otherwise, continue your revision of traits, re-read `Option`, `Result`,
`Iterator` et. al., and then do your homework...

- The tasks will be emailed to you like last time.

---

count: false
class: impact

# Any questions?

???

Obvious any-questionsness
