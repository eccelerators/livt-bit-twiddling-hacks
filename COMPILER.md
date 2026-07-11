# Compiler Notes — Eccelerators.BitTwiddling

## Increment and compound addition are misgenerated in context-free helpers

Using `value++` inside a `public static fn Name[](...)` currently emits no
increment assignment in the generated VHDL. Using `value += amount` emits an
assignment of `amount` instead of an addition. Both forms compile without an
error but produce an incorrect result.

Use an explicit assignment:

```livt
value = value + 1
```

This affects population counts, byte counts, bit-position scans, and one-hot
decoding helpers in this package. Use explicit assignments for all accumulators.

## `out` parameters generate invalid context-free package VHDL

An `out` parameter on `public static fn Name[](...)` currently generates an
assignment to an undeclared output record in the VHDL package body.

The package exposes separate pure result and predicate helpers instead. For
example, use `AddWrapping(a, b)` with `HasAdditionCarry(a, b)`.

All production helpers are static context-free functions grouped in stateless
classes. Keep this file limited to reproducible compiler or generated-VHDL
issues that affect the package. Remove entries when the corresponding issue is
fixed and the full test suite verifies the direct form.
