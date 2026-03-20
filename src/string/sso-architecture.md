# Small String Optimization (SSO)

In standard C, if you want a string that can grow dynamically, you have to use
`malloc()` to allocate memory on the heap.

The problem is that heap allocations are notoriously slow. Requesting memory
from the operating system, finding a contiguous block, and managing pointers
adds significant overhead. If you are processing thousands of short strings—like
parsing names, IP addresses, or configuration keys—this heap allocation
bottleneck will severely degrade your application's performance.

NextStd solves this elegantly using a technique called
**Small String Optimization (SSO)**.

## How SSO Works

The core idea behind SSO is simple: **Why ask the OS for memory if the string is
small enough to fit inside the struct itself?**

When you initialize an `ns_string`, the Rust backend checks the length of the
text you are trying to store.

1. **The Stack Path (Short Strings):** If the string is short (typically under
   15-23 characters, depending on the system architecture), NextStd stores the
   characters *directly inside* the `ns_string` struct on the stack. Zero heap
   allocations are performed.
2. **The Heap Path (Long Strings):** If the string exceeds the inline capacity,
   NextStd automatically falls back to allocating space on the heap and stores a
   pointer to that data inside the struct.

## The Performance Impact

Because most strings in typical applications are short, SSO provides a massive
performance boost:

* **Zero Allocation Cost:** Creating and destroying short strings is practically
  instantaneous.
* **Cache Locality:** Because the string data lives directly inside the struct
  on the stack, the CPU can read it without having to jump across RAM to chase a
  heap pointer. This heavily optimizes CPU cache hits.
* **Safe Fallback:** You never have to manually decide between a fixed-size
  stack array (which risks buffer overflows) and a dynamic heap pointer. The
  `ns_string` handles the transition invisibly.

## Invisible to the User

The best part about NextStd's SSO implementation is that you don't have to think
about it.

Whether your string is 5 characters long and living on the stack, or 50,000
characters long and living on the heap, the C API remains exactly the same. You
still call `ns_string_new` to create it, read from its `data` pointer, and call
`ns_string_free` when you are done.

The Rust backend tracks the memory state internally and guarantees that when
`ns_string_free` is called, it won't accidentally try to `free()` a
stack-allocated string, completely preventing invalid free crashes.
