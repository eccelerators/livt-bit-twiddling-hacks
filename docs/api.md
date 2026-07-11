# API Reference

All `Eccelerators.BitTwiddling` APIs are static context-free helpers. They do
not store state and do not require component construction. Unless a function
explicitly accepts `int` or `byte`, words are unsigned 32-bit `uint` values.

## Integer properties

Namespace: `Eccelerators.BitTwiddling.Integer`

| Function | Result and boundary behavior |
|---|---|
| `SignMask(value)` | `0` for non-negative `int`; `-1` for negative `int` |
| `HaveOppositeSigns(x, y)` | True when one input is negative and the other is non-negative |
| `AbsoluteValueBits(value)` | Unsigned magnitude; represents `abs(INT_MIN)` as `0x80000000` |
| `Minimum(x, y)` | Smaller signed input |
| `Maximum(x, y)` | Larger signed input |
| `IsPowerOfTwo(value)` | True for exactly one set bit; false for zero |
| `RoundUpToPowerOfTwo(value)` | Returns `1` for zero and `0` on 32-bit overflow |
| `SignExtend(value, bitWidth)` | Extends the low 1–32 bits; invalid widths return `0` |

## Masks and fields

Namespace: `Eccelerators.BitTwiddling.Masks`

| Function | Result and boundary behavior |
|---|---|
| `IsAnySet(value, mask)` | True when any selected bit is set |
| `AreAllSet(value, mask)` | True when every selected bit is set; true for an empty mask |
| `Set`, `Clear`, `Toggle` | Updates selected bits and preserves all others |
| `Merge(defaultValue, selectedValue, mask)` | Selects from `selectedValue` where mask bits are one |
| `SetOrClear(value, mask, shouldSet)` | Sets or clears the selected bits |
| `Extract(value, offset, width)` | Right-aligned field; invalid ranges return zero |
| `Insert(value, fieldValue, offset, width)` | Replaced field; invalid ranges preserve `value` |

Offsets count from bit 0, the least-significant bit. Valid fields must fit
entirely inside a 32-bit word.

## Counting and scanning

Namespace: `Eccelerators.BitTwiddling.Counting`

| Function | Result and boundary behavior |
|---|---|
| `PopCount(value)` | Recommended set-bit count, from 0 through 32 |
| `PopCountParallel(value)` | Named parallel educational variant |
| `PopCountIterative(value)` | Kernighan variant with one iteration per set bit |
| `HasOddParity(value)` | True for an odd number of set bits |
| `CountLeadingZeros(value)` | 0–31 for nonzero values; 32 for zero |
| `CountTrailingZeros(value)` | 0–31 for nonzero values; 32 for zero |
| `HighestSetBit(value)` | Highest index from 0–31; `-1` for zero |
| `LowestSetBit(value)` | Lowest index from 0–31; `-1` for zero |
| `Log2Floor(value)` | Highest set-bit index; `-1` for zero |

The iterative APIs are useful as migration references. Consumers that do not
need a specific algorithm should call `PopCount`.

## Bytes

Namespace: `Eccelerators.BitTwiddling.Bytes`

Byte index 0 denotes bits 7 through 0; byte index 3 denotes bits 31 through 24.

| Function | Result and boundary behavior |
|---|---|
| `GetByte(value, index)` | Selected byte; invalid indices return `0x00` |
| `SetByte(value, index, byteValue)` | Updated word; invalid indices preserve `value` |
| `ByteSwap16(value)` | Swapped low two bytes; high 16 result bits are zero |
| `HasZeroByte(value)` | True when any of the four bytes is zero |
| `HasByteEqual(value, expected)` | True when any byte equals `expected` |
| `CountZeroBytes(value)` | Number of zero bytes, from 0 through 4 |

## Permutations and encoding

Namespaces: `Eccelerators.BitTwiddling.Permutation` and
`Eccelerators.BitTwiddling.Encoding`

| Function | Result and boundary behavior |
|---|---|
| `RotateLeft32`, `RotateRight32` | Rotate by 0–31; invalid distances preserve `value` |
| `Reverse32(value)` | Reverses all word bits |
| `ReverseByte(value)` | Reverses all byte bits |
| `ByteSwap32(value)` | Reverses byte order without reversing bits inside bytes |
| `Interleave16(x, y)` | Morton code from the low 16 bits of each coordinate |
| `BinaryToGray(value)` | Reflected 32-bit Gray encoding |
| `GrayToBinary(value)` | Converts reflected Gray code back to binary |
| `IsOneHot(value)` | True for exactly one set bit; false for zero |
| `IsOneHotOrZero(value)` | True for zero or exactly one set bit |
| `IndexToOneHot(index)` | One-hot word for 0–31; invalid indices return zero |
| `OneHotToIndex(value)` | Index for valid one-hot input; otherwise `-1` |

Gray-code conversion is a value transformation, not a complete clock-domain
crossing solution.

## Arithmetic

Namespace: `Eccelerators.BitTwiddling.Arithmetic`

| Function | Result and boundary behavior |
|---|---|
| `ModuloPowerOfTwo(value, power)` | `value mod 2^power`; defined policies for power outside 1–31 |
| `IsDivisibleByPowerOfTwo(value, power)` | Tests the corresponding remainder |
| `AddUnsigned(x, y)` | Saturating addition, clamped to `0xFFFFFFFF` |
| `SubtractUnsigned(x, y)` | Saturating subtraction, clamped to zero |
| `AddWrapping(x, y)` | Modulo-2^32 sum |
| `HasAdditionCarry(x, y)` | True when addition carries beyond bit 31 |
| `SubtractWrapping(x, y)` | Modulo-2^32 difference |
| `HasSubtractionBorrow(x, y)` | True when `y` is greater than `x` |

## Hardware interpretation

These APIs specify value behavior, not a universal area or latency guarantee.
Fixed permutations may reduce mostly to routing, while variable rotations may
infer barrel shifters. Iterative loops, parallel reductions, and comparisons
can produce different circuit structures even when they return the same value.
Inspect synthesis results when timing, throughput, or resource use is part of
the application contract.
