---
layout: post
title: "The Long-Term Impact of LLMs on Software Development"
description: "What will happen to the typical software developer in the years ahead."
date: 2026-09-01 09:00:00 +0000
categories: [general]
tags: [ai, programming]
---

If you've read most any blog from a software developer/engineer/programmer nowdays they'll have a post about where LLMs are taking us long term. Until now, I haven't written about it but I've seen enough to know where this is heading.

The study of software engineering as a discipline will largely be centered on the realm of LLMs (large language models). It'll require fewer people; many working professionals who work on CRUD-oriented systems will eventually be unable to keep their jobs from short sighted management. Many will transition into other adjacent positions, going a level of abstraction higher focusing on planning and organization of the few engineers interacting with frontier LLMs. Think of it as the typewriter pools from the '60s or a computer operators in the early '80s being normalized into tasks we take for granted on a daily bases.

For a majority of my career I've been involved working on CRUD apps and that's where a chunk of my expertise lies, I've made peace with that. After seeing where things are headed for my category of software developer and seeing it continually progress year after year, I've pivoted more into a systems programming track. I'm focusing on C, Go, Rust and Clojure. Now, LLMs can write C just fine as there's a wide selection of source material over many years that's repetitive, but it's more of a lynchpin into BSD and gnu/Linux ecosystem knowledge. Go, Rust and especially Clojure aren't as productive as Python or Javascript for even frontier LLMs.

Why? I could spend hours writing individual examples but I'm going to two glaring cases: 1) Lack of training data and 2)The semantics of the Clojure language.

For Go, correctness depends more on knowing language specific conventions. A model can product perfectly valid Go that is poorly written with bad error handling patterns and unnecessary abstractions. Not to mention misuse of interfaces, poor concurrency structure or failure or use the standard library idiomatically. Also, nowhere near the amount of training data that Python has: The Go team's 2025 developer survey found that most Go developers were using AI development tools, but satisfaction was only middling, with quality concerns specifically called out. Developers also reported wanting help with best practices and making effective use of the standard library. 

With Rust, it requires the model to get much more of the program's semantics right before the code will even compile, while Python has a huge amount of training data and is much more permissive.

Finally, Clojure's semantics are compact but require understanding higher-level concepts like macros, lazy sequences, dynamic state, transactions, et al. As a result, LLMs can often produce valid-looking Clojure that is subtly incorrect, non-idiomatic, or based on patterns borrowed from other languages.

As a result, the select few that work on compilers, low-level networking, kernels, drivers, AI models, et al., shouldn't be as effected by the communization of lower hanging fruit such as CRUD oriented development used for internal tools and similar applications where an established pattern can be followed with Python or JavaScript and the massive training set that those languages have.

There are definitely too many professional developers today, which brings me to the next point. If an LLM can keep track of variables and data structures more efficiently than a human, then why would they be writing software in Python-type scripting languages? Wouldn't you want to use at least a much more efficient language or even just assembly? It seems to me that focusing on something lower lever with a wider record of use, with a breadth of examples to use as training would be a much better use of computing power.

Of course, I could be completely wrong. I'll admit that, if anything, I'm slightly above average in intelligence; the same goes for programming. I'm not looking down on anyone that loses their job due to being unable to compete against these things. It's led me to a bit of an identity crisis, like many others have.

Being able to sit down in front of a minimal text editor and write out a valid program that fulfills a set of requirements is going to be a rare skill. A future programmer will see the code being generated, they'll be able to scan it and understand the individual patterns and make sense of the syntax, the design patterns, and the higher level context of the project. They think "oh yeah, I could do that this is just quicker and allows me to multitask". But when the time comes to demonstrate expertise from first principles, they'll make excuses; They'll jump ship into a new job not facing their past decisions which were short sighted. Even today it's clear that a sizable amount of senior developers can't write coherent code without having a bloated IDE with autocomplete hold their hand. The manifestation of, "Oh yeah I could do that, but I have a Claude" is going to continue to play out year. By going in the opposite direction, it seems one could guarantee being sought after when the time comes to clean up and patch these sysetms.

To be highly sought after, today's programmer should be able to produce something meaningful in the same way that a programmer working in the '80s or '90s would. In that time, it was taught and encouraged for developers to internally picture and map out program design in their head and only then sketch it out more concretely on paper. Computing time was ought-after, a resource that people competed for. It becomes evident that when a person works without many limitations, compared to someone with a numerous hard constraints 

My life and professional experience have shown me that without constraints a person is going to languish in their creativity. With barriers and constraints to work around, that's where creativity and elegant problem solving can flourish. It's why you shouldn't take the autopilot path of having code written for you with an agent such as Claude or Codex. If you can stay reasonably disciplined and even just use AI as high level documentation tool, while all the actual code comes from your consciousness, you'll be several miles ahead of those that took the easy path forward.

I'll see you on the other end of this thing when the sensationalism dies down and the general population hopefully come to their senses.
