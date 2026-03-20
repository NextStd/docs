# Data Structures (`ns_data`)

In standard C, managing collections of data requires constant vigilance.

C provides primitive arrays, which are incredibly fast but notoriously
dangerous. They have a fixed size, they "decay" into raw pointers when passed to
functions (losing all information about their length), and they offer zero
bounds checking. If you accidentally write to `my_array[10]` when the array only
has 5 elements, the compiler won't stop you, leading directly to memory
corruption or a segmentation fault.

Furthermore, standard C lacks built-in advanced data structures altogether. If
you need a dynamically resizing array or a key-value hash map, you either have
to write it yourself from scratch or rely on bloated third-party libraries.

`NextStd` changes this by introducing the `ns_data` module: a suite of
memory-safe, highly performant data structures backed by Rust.

## The NextStd Solution

The data structures in `ns_data` are designed to feel native to C while
providing the airtight memory guarantees of Rust:

* **Dynamic Resizing:** You never have to manually call `realloc()` or calculate
  byte offsets. The Rust backend automatically scales your collections as you
  add or remove elements.
* **Strict Bounds Checking:** If you attempt to access an index or key that
  doesn't exist, NextStd catches it at the FFI boundary and safely returns an
  `ns_error_t` instead of crashing your program.
* **Clean Cleanup:** Just like strings, calling a single `_free` macro on your
  data structure cleanly drops all the heap memory it was managing.

## What's in this chapter?

This chapter breaks down how to initialize, manipulate, and free NextStd's core
collections:

* **[Dynamic Arrays (`ns_vec`)](vectors.md):** Learn how to use memory-safe
  vectors that automatically grow as you push data into them.
* **[Key-Value Maps (`ns_hashmap`)](hashmap.md):** Discover how to store and
  retrieve data using string keys with $O(1)$ average time complexity.
