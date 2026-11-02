---
layout: post
title: "Pointers in Python"
date: 2021-06-21
tags: [python, c, programming, pointers, memory, programming-languages]
---

If you've spent any time with C or C++, you've probably dealt with pointers.

In C, a variable represents a location in memory that contains a value. A pointer is another variable that contains the address of that location.

Python doesn't work that way. There are no C-style pointers in normal Python code.

That doesn't mean Python has no concept of references. It does. In fact, understanding Python's object model makes the whole thing pretty straightforward.

## Python Doesn't Have C-Style Pointers

Python deliberately hides most of the details that C makes you deal with directly.

You normally don't allocate memory yourself. You don't explicitly free it. You don't normally deal with addresses. And you don't have an operator like `&` for taking the address of a variable or `*` for dereferencing a pointer.

Instead, Python has objects and names that refer to those objects.

That's an important distinction.

Consider:

```python
x = 10
```

It's tempting to think of `x` as a box containing `10`.

That's not really what's happening.

Python creates an integer object representing `10`, and the name `x` refers to that object.

You can see the object's identity with `id()`:

```python
x = 10
print(id(x))
```

`id()` gives you an identity value for the object. In CPython, this is typically the object's memory address, but you should not treat `id()` as Python's equivalent of a C pointer.

## Objects and Names

The easiest way to understand Python's model is to stop thinking of Python variables as C variables.

In C:

```c
int x = 10;
```

`x` is an object with storage allocated for an `int`. That storage contains `10`.

In Python:

```python
x = 10
```

`x` is a name bound to an integer object.

Now do this:

```python
y = x
```

Python does not create another integer containing `10`.

Instead, both names refer to the same object:

```python
x = 10
y = x

print(x is y)
```

This produces:

```text
True
```

That's one of the places where Python's object model starts looking vaguely pointer-like.

But it still isn't a C pointer.

There isn't a pointer stored in `y` that you can dereference with `*y`. `y` is simply another name referring to the same Python object.

## Immutable Objects

This distinction becomes particularly obvious with immutable objects.

Integers are immutable.

Consider:

```python
x = 10
y = x

x += 1
```

It might look like Python changed the integer object from `10` to `11`.

It didn't.

The integer object representing `10` is immutable. Instead, Python creates or obtains an integer object representing `11` and binds `x` to that object.

`y` is still referring to the original object:

```python
print(x)
print(y)
```

Output:

```text
11
10
```

You can see the identities change:

```python
x = 10
print(id(x))

x += 1
print(id(x))
```

The important point isn't the actual numbers returned by `id()`. The important point is that `x` is now referring to a different object.

## Mutable Objects

Now consider a list:

```python
x = [10]
y = x
```

Both names refer to the same list object.

If we modify the list:

```python
x[0] = 20
```

then:

```python
print(y)
```

produces:

```text
[20]
```

We didn't make `y` point at a new list.

Both `x` and `y` still refer to the same object. We changed the contents of that object.

This is where mutable Python objects can be used to get behavior that looks somewhat like passing a pointer to a function.

## Simulating Pointer Behavior

Suppose we have this C code:

```c
void add(int *x)
{
    *x += 10;
}
```

The function receives a pointer to an integer and modifies the integer through that pointer.

For example:

```c
int x = 10;

add(&x);
```

After the call, `x` is `20`.

We can get similar behavior in Python by putting the value inside a mutable object:

```python
def add(x):
    x[0] += 10

x = [10]

add(x)

print(x[0])
```

The result is:

```text
20
```

The important thing is what's actually happening here.

Python isn't passing a pointer to an integer.

It's passing a reference to a list object. The function then modifies that list.

That's a useful technique, but calling it a pointer would be misleading.

You can see the difference if you replace the list with a tuple:

```python
x = (10,)
add(x)
```

That fails because tuples are immutable:

```text
TypeError: 'tuple' object does not support item assignment
```

So the trick works because the object is mutable, not because Python suddenly acquired C pointers.

## ctypes: When You Actually Need C Pointers

Python does have a way to work with actual C-compatible pointers.

The standard library module for this is `ctypes`.

This is a completely different situation from using a mutable Python object.

For example, suppose we have a C function:

```c
void add(int *x)
{
    *x += 10;
}
```

We can compile that into a shared library and load it from Python with `ctypes`.

Once loaded, we can create an actual C integer:

```python
import ctypes

x = ctypes.c_int(10)
```

Then we can pass its address to a C function:

```python
add(ctypes.byref(x))
```

After the C function modifies the integer:

```python
print(x.value)
```

We get:

```text
20
```

Here there really is a C pointer involved.

`ctypes.byref(x)` gives the C function access to the address of the `ctypes` object so that the C function can modify the underlying value.

This is fundamentally different from the earlier list example.

The list example was Python using object references and mutability.

The `ctypes` example is Python interoperating with C and actually passing a C-compatible pointer.

## The Important Distinction

There are really three different concepts that tend to get mixed together:

**C variable:** A region of memory containing a value.

**C pointer:** A value containing the address of another object.

**Python name:** A name bound to a Python object.

Python names can behave somewhat like references, but they aren't C pointers.

For ordinary Python programming, you generally don't need to care about memory addresses at all.

If you need something that behaves like a pointer, a mutable object is often enough:

```python
x = [10]
```

If you actually need to interact with C memory and C APIs, then `ctypes` gives you the machinery to do that.

So the short version is:

**Python doesn't have C-style pointers as part of its normal object model. It has objects and names referring to those objects. Mutable objects can be used to achieve some pointer-like behavior, while `ctypes` can be used when you actually need C-compatible pointers.**

That's the distinction worth remembering.
END OF LINE
