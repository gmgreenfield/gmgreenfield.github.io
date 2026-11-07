---
Layout: post
Title: "Starting to Learn Rust Programming"
Description: "Why do I justify starting to learn Rust and an explanation of what makes it good at safety."
date: 2026-11-07 08:26:00 -0500
category: [general]
tags: [rust, programming, distractions]
---

Seeing the exceedingly large amount of exploits constantly discovered in long standing C projects and the fact that I'm seeing opinions of experienced engineers stating, "yeah this is going to be a new constant thing for awhile." has motivated me to start learning Rust.

One thing that I've never understood why so many developers seem to be biased against it for various reasons. Perhaps it could be individuals from the community? Perhaps it's the way that it's introduced to some people? Nevertheless, I've Found an approachable website to start the process, which does't look like it's going to be a short one.

Starting point: https://course.corrode.dev/exercise/00_numbers_in_rust

I must admit before starting to do this, I've been a pretty ardent C traditionalist due to my imprenting with learning it so young (sophmore in high school back in the 00's). However, there are many ways of saying, if your not growing you're dying, and I agree with that as I've observed it with my own eyes. The very reason that I've started this blog has been related to that. To improve writing communication, and demonstrate my curiousity in software development related topics.

The other major reason has been to find new ways to keep my mind off current events and this midterm election. It seems whenever one side looses they have their own special way of attempting to instill their own brand of fear mongering into the American populace. I've debated about if I'd consider writing about my thoughts on that here, but have decided against it.

## Rust and how it achieves Safety

Anyways, regarding rust the core reason of my interest relates to safety: Rust achieves this with ownership and borrowing.

### Ownership

The compiler uses an affine type system to track the ownership of each value: a value can only be used at most once, after which the compiler refuses to use it again.

```
fn main() {
    let original = "Hello, World!".to_string();
    let other = original;
    println!("{}", original);
}
```

Will yield the following error:

```
error[E0382]: use of moved value: `original`
 --> src/main.rs:4:20
  |
3 |     let other = original;
  |         ----- value moved here
4 |     println!("{}", original);
  |                    ^^^^^^^^ value used here after move
  |
  = note: move occurs because `original` has type `std::string::String`, which does not implement the `Copy` trait
```

This, notably, prevents the dreaded double-free regularly encountered in C or C++ (prior to smart pointers).

### Borrowing

The illumination that comes from Rust is that memory issues occur when one mixes aliasing and mutability: that is, when a single piece of memory is accessible through multiple paths and it is mutated (or moved away) leaving behind dangling pointers.

The core tenet of borrow checking is therefore: Mutability XOR Aliasing. It's similar to a Read-Write Lock, in principle.

This means that the Rust compiler tracks aliasing information, for which it uses the lifetime annotations (those 'a in &'a var) to connect the lifetime of references and the value they refer to together.

A value is borrowed if someone has a reference to it or INTO it (for example, a reference to a field of a struct or to an element of a collection). A borrowed value cannot be moved.

Mutability (without aliasing)

You can obtain only a single mutable reference (&mut T) into a given value at any time, and no immutable reference into this value may exist at the same time; it guarantees that you have exclusive access to this tidbit of memory and thus you can safely mutate it.

Aliasing (without mutability)

You can obtain multiple immutable references (&T) into a given value at any time. However you cannot mutate anything through those references (*).

(*) I am lying; there are structs like RefCell which implement "interior mutability"; they do respect the Mutability XOR Aliasing principle, but defer the check to run-time instead.

That's it?

Nearly so ;)

It is already quite complicated to implement for the compiler-writers, and may unduly constrain the users (some programs that would be safe cannot be proven safe using this system, requiring to jump through hoops), however the core principles are really that simple.

### So what's left?

Bounds-checking. It is not rocket-science, but may induce a performance penalty though. Most languages have some degree of support for it, C being the big exception, and C++ having some support for it, although it is optional. [#1]

References:

[1] https://stackoverflow.com/questions/36136201/how-does-rust-guarantee-memory-safety-and-prevent-segmentation-faults

END OF LINE
