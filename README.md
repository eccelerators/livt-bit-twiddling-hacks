# Eccelerators.BitTwiddling

[![Livt](https://img.shields.io/badge/Livt-library-6f42c1)](https://github.com/eccelerators)
[![Package version](https://img.shields.io/badge/version-0.1.0-blue)](livt.toml)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Target: FPGA](https://img.shields.io/badge/target-FPGA-orange)](#hardware-perspective)

Readable, reusable Livt utilities and adaptations of selected algorithms from Sean Eron
Anderson's [Bit Twiddling Hacks](https://graphics.stanford.edu/~seander/bithacks.html).

The original examples are primarily C techniques evaluated using a CPU-oriented
operation count. This project is a migration guide, not a claim that fewer C
operators produce better FPGA hardware. Each implementation makes its 32-bit
width, signedness, input constraints, and boundary behavior explicit. Clear
Livt code is preferred over preserving a branchless or compact C expression.

According to the original page, its individual snippets are in the public domain
unless otherwise noted. The aggregate page and descriptions remain copyrighted
by their author. This repository contains original Livt adaptations and links to
the source instead of reproducing the article.

## 🧭 Design principles

- Prefer descriptive APIs and intermediate values over compressed expressions.
- Use `uint` for raw 32-bit words and `int` only for signed semantics.
- Define zero, overflow, invalid-width, and shift-boundary behavior.
- Use ordinary control flow when it communicates the intended circuit clearly.
- Keep straightforward and specialized implementations separately named.
- Treat FPGA area, latency, throughput, and combinational depth as separate
  concerns from CPU instruction counts.

## 📦 Package API shape

The package exposes stateless classes with public static context-free helpers.
Callers use APIs such as `BitCounting.PopCount(value)` and
`BitMasks.Extract(value, offset, width)` directly; no component instance or
scheduled dispatcher is required.

The ordinary operation names are the recommended reusable entry points. Names
that describe an implementation strategy exist for comparison and migration
education. For example, `PopCount` is the normal API, while
`PopCountIterative` and `PopCountParallel` expose the two source structures.
Callers should depend on the ordinary API unless they intentionally require a
particular implementation.

All APIs use explicit 32-bit `uint` or `int` semantics. Functions are grouped by
developer intent so applications can reuse field manipulation, encoding,
scanning, byte access, and arithmetic helpers without depending on the history
of the corresponding C trick.

## 🧰 Public classes

| Class | Namespace | Responsibilities |
|---|---|---|
| `IntegerProperties` | `Eccelerators.BitTwiddling.Integer` | Sign, magnitude, min/max, powers of two, sign extension |
| `BitMasks` | `Eccelerators.BitTwiddling.Masks` | Merge, conditional set/clear, field extraction and insertion |
| `BitCounting` | `Eccelerators.BitTwiddling.Counting` | Iterative and parallel population count, parity |
| `BitScanning` | `Eccelerators.BitTwiddling.Counting` | Integer log2 and significant-bit scans |
| `BitPermutation` | `Eccelerators.BitTwiddling.Permutation` | Word/byte reversal, rotation, and byte swapping |
| `MortonCodes` | `Eccelerators.BitTwiddling.Permutation` | Interleaving two 16-bit coordinates |
| `BytePredicates` | `Eccelerators.BitTwiddling.Bytes` | Zero-byte counting and byte equality |
| `ByteOperations` | `Eccelerators.BitTwiddling.Bytes` | Indexed byte extraction, replacement, and 16-bit byte swap |
| `WordEncoding32` | `Eccelerators.BitTwiddling.Encoding` | Gray code and one-hot conversion/validation |
| `SaturatingArithmetic` | `Eccelerators.BitTwiddling.Arithmetic` | Unsigned saturation, wrapping results, carry, and borrow predicates |
| `Modulus` | `Eccelerators.BitTwiddling.Arithmetic` | Remainder and divisibility by powers of two |

## 🔄 Compatibility matrix

`Direct` means the identity maps clearly to Livt. `Adapted` means C-specific
semantics were replaced with explicit Livt behavior. `Alternative` means the
repository uses a clearer hardware-oriented formulation. `Unsupported` means
the original depends on features that do not have a direct Livt equivalent.

| Original hack family | Migration | FPGA character | Project treatment |
|---|---|---|---|
| Sign and opposite-sign detection | Direct | General bit logic | Implemented |
| Branchless absolute value | Adapted | General arithmetic | Implemented with a documented `INT_MIN` result |
| Branchless min/max | Alternative | Comparator and mux | Implemented with readable `if` statements |
| Power-of-two detection | Direct | AND, subtractor, zero detector | Implemented |
| Sign extension | Adapted | Masking or sign-bit wiring | Implemented with width validation |
| Conditional set/clear | Alternative | Per-bit mux/mask | Implemented with readable control flow |
| Merge according to a mask | Direct | Per-bit mux | Implemented |
| Population count | Direct | Iterative circuit or adder tree | Both readable variants implemented |
| Parity | Direct | XOR reduction tree | Implemented |
| Lookup-table count/parity | Supported, omitted | ROM or mux network | Direct logic is clearer here |
| XOR or add/subtract swap | Omitted | CPU-specific trick | A temporary value is clearer in Livt |
| Swap individual bit ranges | Adapted | Masking and routing | Deferred until a reusable validated API is needed |
| Bit reversal | Direct | Fixed routing or staged masks | Implemented |
| Modulo by `2^n` | Direct | Mask or wire selection | Implemented |
| Modulo by `2^n - 1` | Supported, deferred | Reduction network | Not yet implemented |
| Integer log2 / highest set bit | Alternative | Priority encoder | Implemented as readable binary search |
| Trailing-zero count | Alternative | Priority encoder | Implemented as readable binary search |
| Leading-zero count | Alternative | Priority encoder | Implemented as readable binary search |
| Highest/lowest set bit | Alternative | Priority encoder | Implemented with explicit sentinel behavior |
| Round up to a power of two | Direct | Shift/OR network | Implemented with overflow behavior |
| Rotate left/right | Direct | Fixed wiring or barrel shifter | Implemented with validated distances |
| Morton interleaving | Direct | Fixed permutation/masking | Implemented |
| Zero-byte and equal-byte tests | Alternative | Parallel byte comparators | Implemented explicitly |
| Next bit permutation | Supported, deferred | Arithmetic and priority logic | Not yet implemented |
| Pointer-based byte access | Unsupported | C layout/aliasing dependent | Use masks, shifts, arrays, or slices |
| C bit fields, unions, templates, macros | Unsupported | C language dependent | Use explicit Livt types and helpers |
| Float representation tricks | Unsupported directly | IEEE-754-specific datapath | Requires a dedicated vector-level float design |
| 64-bit multiply tricks | Unsupported by `uint` | Wide multiplier/modulus circuit | Prefer 32-bit alternatives |
| CPU branch-avoidance tricks | Simplified | Branches normally become muxes | Prefer readable Livt control flow |

## 🧩 Additional Livt utilities

Some reusable operations are not taken directly from the Stanford collection
but belong naturally beside the migrated algorithms:

- Indexed byte access and replacement for memory words and protocol fields.
- Binary/Gray conversion for counters and encoders. Gray encoding alone does
  not make a clock-domain crossing safe; synchronization is still required.
- One-hot validation and index conversion for arbiters and control logic.
- Saturating and wrapping unsigned arithmetic with carry/borrow predicates.
- Validated rotations for ALUs, checksums, hashes, and serializers.

## ⚙️ Hardware perspective

A mathematical identity can migrate correctly without remaining an
optimization. For example, a branchless CPU minimum is naturally a comparator
and multiplexer in an FPGA, while fixed bit reversal may reduce mostly to
routing. Population count can be a small iterative circuit, a combinational
adder tree, or a pipelined tree depending on system requirements.

The implementations here are behavioral references. Synthesis comparisons may
later add area, latency, throughput, inferred-memory, and critical-path results
without changing these public APIs.

## 🗂️ Layout

```text
src/integer/      signed and unsigned integer properties
src/masks/        masks and bit-field manipulation
src/counting/     population count and significant-bit scans
src/permutation/  rotations, reversal, byte swaps, and Morton codes
src/bytes/        indexed byte operations and byte predicates
src/arithmetic/   power-of-two modulus and saturating arithmetic
src/encoding/     Gray-code and one-hot helpers
tests/<domain>/   tests mirroring each source domain
docs/             short package usage examples
```

Production namespaces mirror their folders, for example:

```livt
namespace Eccelerators.BitTwiddling.Counting
```

Tests mirror the same domain below `Eccelerators.BitTwiddling.Tests`, for example:

```livt
namespace Eccelerators.BitTwiddling.Tests.Counting

using Eccelerators.BitTwiddling.Counting
```

## 🛡️ Defined boundary behavior

- `AbsoluteValueBits(-2147483648)` returns `0x80000000` as `uint`.
- `RoundUpToPowerOfTwo(0)` returns `1`.
- `RoundUpToPowerOfTwo` returns `0` when the result exceeds 32 bits.
- `SignExtend` returns `0` for widths outside 1 through 32.
- `Log2Floor(0)` returns `-1`.
- `CountTrailingZeros(0)` returns `32`.
- `CountLeadingZeros(0)` returns `32`.
- `HighestSetBit(0)`, `LowestSetBit(0)`, and invalid one-hot decoding return `-1`.
- Invalid rotation distances preserve the input value.
- Invalid byte indices return zero when reading and preserve the word when writing.
- Invalid `Extract` requests return `0`; invalid `Insert` requests preserve the
  original value.
- `ModuloPowerOfTwo(value, 0)` returns `0`; powers of 32 or more return `value`.

## ✅ Build and test

```sh
livt build
livt test
```

The package metadata enables all compiler warnings for debug and release builds.
Short integration examples are available in [`docs/usage.md`](docs/usage.md).

## 🛠️ Development notes

- Keep stateless reusable operations as static context-free methods on classes.
- Keep source and tests in matching domain folders and namespaces.
- Add boundary tests for zero, maximum values, invalid widths, and invalid indices.
- Record only active reproducible compiler workarounds in `COMPILER.md`.
- Preserve the readable public API even when adding educational alternatives.

## 📄 License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE).
