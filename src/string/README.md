# Strings (`ns_string`)

In standard C, a string is nothing more than a contiguous array of characters in
memory, terminated by a null byte (`\0`).

While this design is incredibly minimalist, it is also the root cause of
countless security vulnerabilities. Because standard C strings do not inherently
know their own length or capacity, functions like `strcpy` and `strcat` will
happily overwrite adjacent memory if the destination buffer is too small,
leading to catastrophic **buffer overflows**.

Furthermore, simply finding the length of a standard C string using `strlen()`
requires an `O(N)` operation—scanning every single byte until the null
terminator is found.

## The NextStd Solution

The `ns_string` module introduces a radically different approach. Instead of raw
character pointers, NextStd strings are intelligent structs managed safely by
the Rust backend.

By tracking their own length and capacity, NextStd strings make out-of-bounds
writes architecturally impossible. If you try to append text that exceeds the
current capacity, the Rust backend will automatically and safely reallocate the
heap memory for you.

Even better, `ns_string` implements **Small String Optimization (SSO)**. This
means short strings avoid the slow, expensive heap allocation process entirely
and are stored directly on the stack, delivering blistering performance that
rivals or beats standard C arrays.

## What's in this chapter?

This chapter breaks down how to create, manage, and manipulate memory-safe
strings:

- **[SSO Architecture](sso-architecture.md):** Look under the hood at how
  NextStd avoids heap allocations for short strings to maximize performance.
- **[String Lifecycle](lifecycle.md):** Learn how to initialize, read, and
  safely free `ns_string` objects.
- **[Concatenation & Manipulation](concatenation.md):** Discover how to safely
  append and modify strings without ever worrying about buffer overflows again.
