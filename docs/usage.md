# Usage Examples

`Eccelerators.BitTwiddling` exposes stateless, fixed-size 32-bit helpers. All public
operations are static and context-free, so callers do not construct component
instances and can compose the returned values directly.

## Count and scan bits

```livt
namespace Example

using Eccelerators.BitTwiddling.Counting

component CountExample
{
    public fn Run()
    {
        var setBits: int = BitCounting.PopCount(0xF0F00001)
        var highest: int = BitScanning.HighestSetBit(0x00010000)
        var leadingZeros: int = BitScanning.CountLeadingZeros(0x00010000)
    }
}
```

`HighestSetBit(0)` returns `-1`. Leading- and trailing-zero counts return `32`
for zero.

## Extract and replace fields

```livt
namespace Example

using Eccelerators.BitTwiddling.Masks

component FieldExample
{
    public fn Run()
    {
        var word: uint = 0x12345678
        var field: uint = BitMasks.Extract(word, 8, 8)
        var replaced: uint = BitMasks.Insert(word, 0xAB, 8, 8)
    }
}
```

Offsets count from the least-significant bit. Invalid extraction requests
return zero; invalid insertion requests preserve the original word.

## Work with bytes

```livt
namespace Example

using Eccelerators.BitTwiddling.Bytes

component ByteExample
{
    public fn Run()
    {
        var word: uint = 0x12345678
        var second: byte = ByteOperations.GetByte(word, 1)
        var changed: uint = ByteOperations.SetByte(word, 1, 0xAB)
        var containsZero: bool = BytePredicates.HasZeroByte(changed)
    }
}
```

Byte index zero identifies the least-significant byte.

## Encode and permute values

```livt
namespace Example

using Eccelerators.BitTwiddling.Encoding
using Eccelerators.BitTwiddling.Permutation

component EncodingExample
{
    public fn Run()
    {
        var gray: uint = WordEncoding32.BinaryToGray(42)
        var binary: uint = WordEncoding32.GrayToBinary(gray)
        var rotated: uint = BitPermutation.RotateLeft32(binary, 5)
        var morton: uint = MortonCodes.Interleave16(12, 7)
    }
}
```

Gray-code conversion does not by itself make a clock-domain crossing safe.

## Arithmetic boundaries

```livt
namespace Example

using Eccelerators.BitTwiddling.Arithmetic
using Eccelerators.BitTwiddling.Integer

component ArithmeticExample
{
    public fn Run()
    {
        var saturated: uint = SaturatingArithmetic.AddUnsigned(0xFFFFFFFF, 1)
        var remainder: uint = Modulus.ModuloPowerOfTwo(0x12345678, 8)
        var rounded: uint = IntegerProperties.RoundUpToPowerOfTwo(1000)
    }
}
```

See the package README for all defined sentinel and invalid-input behavior.
