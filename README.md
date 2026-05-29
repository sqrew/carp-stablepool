# carp-stablepool

A high-performance, generational object pool (arena) for the [Carp language](https://github.com/carp-lang/Carp).

This module provides $O(1)$ allocation and deallocation of objects while providing safe, stable references via generational handles.

## Features

- **Generational Safety**: Uses `carp-handle` with `Uint32` generation counters to ensure stale references are caught with zero ambiguity.
- **High Performance**: $O(1)$ constant-time allocation and deallocation using an internal free-list.
- **Memory Efficient**: Backed by a contiguous array to minimize fragmentation, with a tiered slot model that separates occupancy from generation state.
- **Robust Invariants**: Strictly maintains length and free-list state, with program aborts for critical internal violations (e.g. double-free).
- **Type-Safe**: Generic implementation allows pooling any Carp type.
- **Zero Dependencies**: Pure Carp implementation requiring only `carp-handle`.

## Design Philosophy

The StablePool is designed for foundation-grade resource management:

1. **Stable Indices**: Unlike a regular array, removing an element does not shift others. Handles remain valid until the object is explicitly freed.
2. **Generational Hardening**: Every slot tracks its generation using an unsigned 32-bit counter. Comparisons are exact and wraparound is documented.
3. **Safety Layering**: Isoles `Array.unsafe-nth` behind internal checked boundaries. Random access via handles is fully validated.
4. **Pragmatic Utilities**: Provides O(n) iteration and reduction, and a debug-only `contains?` helper.

## Installation

Add this to your project by loading `stable_pool.carp`.

```clojure
(load "path/to/carp-handle/handle.carp")
(load "path/to/carp-stablepool/stable_pool.carp")
(use StablePool)
```

## Usage

```clojure
(use StablePool)

(let [pool (StablePool.new)]
  (do
    (let [h (StablePool.alloc! &pool @"bullet")]
      (do
        (IO.println &(str (StablePool.get &pool &h))) ; (Just "bullet")
        (StablePool.free! &pool &h)
        (IO.println &(str (StablePool.get &pool &h))))) ; Nothing
    ))
```

## Running Tests

```bash
carp -x test/stable_pool_test.carp
```

## License

MIT
