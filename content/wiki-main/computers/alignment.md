+++
title = "Memory Accessing Alignment"
draft = false
date = "2017-04-12"
wikis = ["computers"]
tags = ["c", "lowlevel", "performance"]
+++

For any data type that requires `N` bytes, its starting address should be a
multiple of `N`. In most `x86` processors the memory interface is designed to
read/write blocks that are 8 or 16 bytes long<sup>2</sup>.

> *Unaligned memory accesses* occur when you try to read `N` bytes of data
> starting from an address that is not evenly divisible by `N` (i.e. `addr % N
> != 0`).  For example, reading 4 bytes of data from address `0x10004` is fine,
> but reading 4 bytes of data from address `0x10005` would be an unaligned
> memory access <sup>6</sup>.

Unaligned access may allow the program to use less
memory<sup>1</sup>, but it has many drawbacks:


> The effects of performing an unaligned memory access vary from architecture
> to architecture. A summary of the common scenarios:
>
> * Some architectures are able to perform unaligned memory accesses
>   transparently, but there is usually a significant performance cost.
> * Some architectures raise processor exceptions when unaligned accesses
>   happen. The exception handler is able to correct the unaligned access, at
>   significant cost to performance.
> * Some architectures raise processor exceptions when unaligned accesses
>   happen, but the exceptions do not contain enough information for the
>   unaligned access to be corrected.
> * Some architectures are not capable of unaligned memory access, but will
>   silently perform a different memory access to the one that was requested,
>   resulting in a subtle code bug that is hard to detect!
>
> If your code causes unaligned memory accesses to happen, your code will not
> work correctly on certain platforms and will cause performance problems on
> others <sup>6</sup>.

## Example

Consider a bitmap data structure where each pixel is represented by 3 bytes (RGB). In order to preserve alignment we add a "padding byte"<sup>3</sup>, making the structure 32 bits instead of 24:

```
+---------------+---------------+---------------+---------------+
| : : : : : : : | : : : : : : : | : : : : : : : | : : : : : : : |
+---------------+---------------+---------------+---------------+
     Red             Green           Blue            Padding
```

This consumes more memory, but is ideally more performant. Of course, YMMV<sup>5</sup>.

---

<small>

  1. *Introduction To ARM Cortex-M Microcontrollers* (5<sup>th</sup> Ed.), p. 96.
  2. *Computer Systems: A Programming Perspective* (2<sup>nd</sup> Ed.), p. 290.
  3. [*Handmande Hero, Day 004*](https://youtu.be/hNKU8Jiza2g?t=5m19s) (t. 5:19).
  4. *21<sup>st</sup> Century C*, (2<sup>nd</sup> Ed.), p. 137.
  5. [Data Alignment For Speed: Myth Or Reality?](http://lemire.me/blog/2012/05/31/data-alignment-for-speed-myth-or-reality/).
  6. [Kernel.org: Unaligned Memory Access](https://www.kernel.org/doc/Documentation/unaligned-memory-access.txt).

</small>
