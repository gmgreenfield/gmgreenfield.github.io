---
layout: post
title: "Starting to Learn Rust Programming"
description: "Why I decided to start learning Rust and an explanation of what makes it good at safety."
date: 2026-11-07 08:26:00 -0500
category: [general]
tags: [rust, programming, distractions]
---

Seeing the exceedingly large amount of exploits constantly discovered in long-standing C projects and the fact that I'm seeing opinions of experienced engineers stating, "yeah, this is going to be a constant thing for a while," has motivated me to start learning Rust.

One thing that I've never understood is why so many developers seem to be biased against it for various reasons. Perhaps it could be individuals from the community? Perhaps it's the way that it's introduced to some people? Nevertheless, I've found an approachable website to start the process, which doesn't look like it's going to be a short one.

Starting point: [Rust exercises](https://course.corrode.dev/exercise/00_numbers_in_rust)

I must admit before starting to do this, I've been a pretty ardent C traditionalist due to my imprinting with learning it so young (sophomore in high school back in the 00s). However, there's something to be said for the idea that if you're not growing, you're dying, and I agree with that as I've observed it with my own eyes. The very reason that I've started this blog has been related to that: to improve my written communication and demonstrate my curiosity in software development-related topics.

The other major reason has been to find new ways to keep my mind off current events and this midterm election. It seems whenever one side loses, they have their own special way of attempting to instill their own brand of fearmongering into the American populace. I've debated whether I'd consider writing about my thoughts on that here, but have decided against it.

## Rust and How It Achieves Safety

Anyways, regarding Rust, the core reason for my interest relates to safety: Rust achieves this through ownership and borrowing.

### Ownership

The compiler uses an affine type system to track the ownership of each value: a value can only be used at most once after ownership has been moved, after which the compiler refuses to allow further use of the original binding.

```rust
fn main() {
    let original = "Hello, World!".to_string();
    let other = original;
    println!("{}", original);
}
```

This will yield the following error:

```text
error[E0382]: borrow of moved value: `original`
 --> src/main.rs:4:20
  |
3 |     let other = original;
  |         ----- value moved here
4 |     println!("{}", original);
  |                    ^^^^^^^^ value borrowed here after move
  |
  = note: move occurs because `original` has type `std::string::String`, which does not implement the `Copy` trait
```

This, notably, prevents double-free errors caused by multiple owners attempting to free the same allocation, a problem regularly encountered in C or C++ when ownership is not managed correctly.

### Borrowing

The illumination that comes from Rust is that memory issues occur when one mixes aliasing and mutability: that is, when a single piece of memory is accessible through multiple paths and it is mutated (or moved away), potentially leaving behind dangling pointers.

The core tenet of borrow checking is therefore: Mutability XOR Aliasing. It's similar to a Read-Write Lock, in principle.

This means that the Rust compiler tracks aliasing information, using lifetime annotations (those `'a` in `&'a T`) to connect the lifetimes of references and the values they refer to.

A value is borrowed if someone has a reference to it or INTO it (for example, a reference to a field of a struct or to an element of a collection). A borrowed value cannot be moved while that borrow remains active.

**Mutability (without aliasing)**

You can obtain only a single mutable reference (`&mut T`) into a given value at any time, and no immutable reference into this value may exist at the same time; it guarantees that you have exclusive access to this tidbit of memory and thus you can safely mutate it.

**Aliasing (without mutability)**

You can obtain multiple immutable references (`&T`) into a given value at any time. However, you cannot mutate anything through those references (*).

(*) I am lying; there are structs like `RefCell` which implement "interior mutability"; they do respect the Mutability XOR Aliasing principle, but defer the check to runtime instead.

### That's It?

Nearly so ;)

It is already quite complicated to implement for compiler writers, and may unduly constrain users (some programs that would be safe cannot be proven safe using this system, requiring you to jump through hoops). However, the core principles really are that simple.

### So What's Left?

Bounds checking. It is not rocket science, but may induce a performance penalty. Most languages have some degree of support for it, C being the big exception, and C++ having some support for it, although it is optional. [1]

## Practicing Algorithms in Rust

Tying this into practicing leet code style problems, fortunately [LeetCode](https://leetcode.com/problemset/) supports Rust directly, so there's no need to translate solutions from another language or maintain a separate development environment just to work through problems.

There's also [Exercism's Rust track](https://exercism.org/tracks/rust), which takes a somewhat different approach. Rather than focusing primarily on interview problems, it provides language-specific exercises intended to help develop familiarity with Rust's syntax and idioms.

The two approaches complement each other nicely. LeetCode provides the algorithmic practice, while Exercism provides an opportunity to become more comfortable writing Rust itself. In particular I don't want to simply learn enough Rust to translate solutions from Python or C, I'd like to develop an understanding of how Rust programmers actually approach problems.

## References

[1] [How does Rust guarantee memory safety and prevent segmentation faults?](https://stackoverflow.com/questions/36136201/how-does-rust-guarantee-memory-safety-and-prevent-segmentation-faults)

END OF LINE
