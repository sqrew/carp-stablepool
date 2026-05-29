# carp-stablepool

A high-performance, generational object pool (arena) for the [Carp language](https://github.com/carp-lang/Carp).

This module provides $O(1)$ allocation and deallocation of objects while providing safe, stable references via generational handles.

## Features

- **Generational Safety**: Uses `carp-handle` to ensure that stale references to deleted objects cannot access new objects in the same slot.
- **High Performance**: $O(1)$ constant-time allocation and deallocation using an internal free-list.
- **Memory Efficient**: Backed by a contiguous array to minimize fragmentation.
- **Type-Safe**: Generic implementation allows pooling any Carp type.
- **Zero Dependencies**: Pure Carp implementation requiring only `carp-handle`.

## Design Philosophy

The StablePool is designed for massive entity counts (e.g. bullets, particles, ECS components):

1. **Stable Indices**: Unlike a regular array, removing an element does not shift others. Handles remain valid until the object is explicitly freed.
2. **Detection**: Every slot tracks its \"generation\". Handles include this counter, allowing the pool to detect if a reference is stale.
3. **Pragmatic**: Automatically grows when full, maintaining its performance guarantees.

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
