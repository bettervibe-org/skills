---
name: better-rust
description: A skill on how to write good Rust code.
---

The meta-principle: good Rust code trusts the type system and borrow checker to enforce correctness, rather than working around them. If you find yourself constantly fighting the compiler, it's usually a signal to rethink your design — not find a workaround.

## Ownership & Borrowing
*Good*: Works with the borrow checker by designing clear ownership hierarchies, using references appropriately, and keeping lifetimes short.
*Bad*: Fights the borrow checker — reaching for `clone()` everywhere, overusing `Rc<RefCell<T>>`, or scattering `Arc<Mutex<T>>` without justification. Cloning is sometimes correct, but reflexive cloning to silence the compiler is a smell.

## Type System Usage
*Good*: Encodes invariants in types so illegal states are unrepresentable. Uses newtypes to distinguish semantically different values (e.g., `Miles(f64)` vs `Kilometers(f64)`). Leverages enums with data to model state machines.
*Bad*: Uses primitive types for everything, relies on runtime checks and assertions instead of compile-time guarantees, or uses boolean flags where a two-variant enum would be clearer.

## Iterators vs. Loops
*Good*: Uses iterator chains (map, filter, fold, collect) for transformations — they're often clearer and as fast or faster due to compiler optimization.
*Bad*: Writes verbose index-based for loops with manual accumulation where a clean iterator chain would do.

## Error Handling
*Good*: Uses `Result<T, E>` and `Option<T>` idiomatically. Propagates errors with ?, defines meaningful custom error types (often via thiserror), and never silently swallows errors.
*Bad*: Sprinkles `.unwrap()` and `.expect()` liberally in production paths, uses panic! for recoverable errors, or returns stringly-typed errors everywhere.
