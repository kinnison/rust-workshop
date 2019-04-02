title: Rust Workshop - Session 4
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
  - Learn about more ways to organise code
  - Look at a couple of crates

---

title:
class: middle

# Topics

???

1. Going over homework (brief only, we did this last week in depth)
2. Modules - what, how, where, why?
3. Command line handling with `clap`
4. Command line handling with `structopt`

---

title:
class: impact

# Handing in Homework

???

Let's go over your homework questions and ways in which you might have
solved them.

---

title: Modules

- Why?

???

- We've seen crates as a way to separate out code
- But what if you have a huge amount of code to hold in one crate?
- Then we need modules

---

title: Modules - the filesystem

```
\ src
    + lib.rs
    + foo.rs
    \ bar/mod.rs
```

???

Any submodule exists in one of two files, either `foo.rs` for `mod foo` or
else `foo/mod.rs`.

---

title: Modules - in code

```rust
mod foo;

mod bar {
    // Some code
}
```

???

If you want to have a module, you have to tell the compiler where the code can
be found for the module. This is done with the `mod` syntax.

You can either say `mod foo;` and put the content in an external file like the
previous slide showed, or you can say `mod bar { ... }` and put the code inside
the module file you already have, like the test code we've done in the past
couple of weeks.

---

title: Modules - Conditional compilation

```rust
#[cfg(test)]
mod stuff {
	... // mock implementation
}

#[cfg(all(not(test), target_os = "windows"))]
mod stuff {
    ... // windows impl (maybe calls win32?)
}

#[cfg(all(not(test), not(target_os = "windows"))]
mod stuff {
    ... // unixy impl (maybe calls posix?)
}

// And bring it into scope
pub use self::stuff::*;
```

???

You can put conditions on module statements to allow them to be included
only in certain circumstances.  We've seen this with `#[cfg(test)]` but it can
also be used to bring in different implementations (for example Windows vs
Linux impls of stuff)

---

title: Modules - Conditional compilation with external files

```rust
use cfg_if::cfg_if;

cfg_if! {
    if #[cfg(unix)] {
        mod unix;
        pub use self::unix::*;
    } else if #[cfg(windows)] {
        mod windows;
        pub use self::windows::*;
    } else {
        compile_error!("Unable to find implementation")
    }
}
```

???

You can also use external module files, though that gets a smidge more
complicated since we have to conditionalise the import too

---

title: Moving on
class: impact

# Moving on

???

Now let's look at some libraries for doing something useful.  We'll need one of
these, or something similar, to complete the homework this week…

---

title: Commandline handling - Clap

- Proper command line parsing is important
- We've been doing `std::env::args().skip(1).next()` and other horrors
- Let's look at a nicer way to do this

???

Clap is a good command line handler which is designed to produce nice command
line APIs with good help etc for minimum effort.

Clap uses the builder pattern which we discussed in the previous proper session.

---

title: Commandline handling - Clap - Simple example

```rust
App::new("Brainfuck Tool")
     .version(env!("CARGO_MANIFEST_VERSION"))
     .author("Daniel Silverstone")
     .about("Interprets Brainfuck programs")
     .arg(Arg::with_name("PROGRAM")
          .help("The program to interpret")
          .required(true)
     .get_matches()
```

???

Here's a very simple Clap setup which is for our `bft` program. It returns a
`matches` object.

---

title: Commandline handling - Clap - Using the matches

```rust
matches.value_of("PROGRAM").unwrap() // Safe because .required(true)

matches.value_of("PROGRAM")                        // Nicer,
       .expect("BUG: mandatory PROGRAM not found") // but unnecessary
```

???

While this looks like more code than we had before (and it is) this is better
than using `env::args()` because we can more cleanly extend it going forward.

---

title: Clap in action
class: impact

# Clap in action

???

Let's create a new binary crate and have a go with clap.

---

title: Commandline handling - Structopt - Equivalence

```rust
/// A Brainfuck tool
#[derive(StructOpt,Debug)]
#[structopt(name = "bft")]
struct Options {
    /// The program to interpret
    #[structopt(name="PROGRAM", parse(from_os_str))]
    program: PathBuf,
}

fn main() {
    let opt = Options::from_args();
    println!("{:?}", opt);
}
```

???

The `structopt` crate actually uses `clap` under the hood, but allows you to
define your options by means of a structure instead of open-coding them.

I'm not going to state which one you should use, not least because I use both
in different circumstances.  If you do choose `clap` be aware there's many
different ways to use it, the builder pattern shown today, but also a macro,
and a YAML approach.

---

title: Homework

Play with `structopt` and `clap` because you'll need one of them (or something
similar) for your homework.  You'll also want to revise `PathBuf` and `Path`,
and you might want to learn about `OsStr` and `OsString` because they might
be important depending on how you do your homework.

Research bracket matching algorithms because, like the above, you'll need one.

If you run out of things to do, then research `std::io::Cursor` because we'll
be using it next week.

- The tasks will be emailed to you like last time.

---

count: false
class: impact

# Any questions?

???

Obvious any-questionsness
