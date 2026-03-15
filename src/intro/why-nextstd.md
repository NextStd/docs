# Why NextStd?

C is the undisputed language of the physical and systems world.
Whether you are building real-time camera processing pipelines,
writing constrained firmware for microcontrollers like the ESP32
and RP2040, or crafting lightning-fast terminal applications, C gives
you the raw, uncompromised control you need.

But that control comes with a massive, decades-old hidden cost:
**The Standard Library.**

## The Legacy C Trap

Libraries like `<stdio.h>`, `<string.h>`, and `<stdlib.h>` were designed
in an era before cybersecurity was a primary concern. Today, they are
the root cause of the majority of software vulnerabilities.

Consider a standard C input operation:

```c
int age;
printf("Enter your age: ");
scanf("%d", &age);
```

If a user types `"twenty"` instead of `20`, `scanf` silently fails,
leaves the invalid text in the input buffer (corrupting future reads),
and leaves `age` uninitialized. If you try to concatenate two strings using
`strcat` and miscalculate the buffer size by a single byte, you trigger a
buffer overflow. If you pass a `NULL` pointer into `strlen`, your entire
system crashes with a Segmentation Fault.

In mission-critical environments, a single uncaught `NULL` pointer can brick a
device or crash a production system.

### The Rewrite Dilemma

The modern industry answer to C's unsafety is to rewrite everything in Rust.

Rust is incredible. It mathematically proves memory safety at compile time.
However, rewriting massive, established C codebases, or trying to interface
Rust with highly specific hardware vendor SDKs, is often unrealistic, expensive,
and time-consuming.

Developers shouldn't have to abandon their C codebases just to get modern safety
guarantees.

## The NextStd Bridge

**NextStd** was built to be the bridge. It is a drop-in systems library that
brings the fearless safety of Rust directly into C, using standard Foreign
Function Interface (FFI) boundaries.

With NextStd, you don't rewrite your C code; you just upgrade your tools.

* **You keep** your C compiler, your Makefiles, and your existing architecture.
* **You replace** dangerous functions like `printf` and `scanf` with
`ns_print` and `ns_read`.
* **You gain** compile-time type routing, `NULL` pointer immunity,
safe heap allocation, and Python-style `TRY/EXCEPT` error handling.

By handling the most dangerous operations (I/O, string manipulation,
and dynamic memory) in a memory-safe backend, NextStd allows you to write
C code that is fundamentally immune to standard library footguns.

You get the speed of C, with the peace of mind of Rust.
