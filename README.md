# unsigned-varint

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](https://protocol.ai)
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)](https://github.com/multiformats/multiformats)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](https://webchat.freenode.net/?channels=%23ipfs)
[![](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> unsigned varint in use in multiformat specs

This unsigned varint (VARiable INTeger) format is for the use in all the multiformats.

## Format

Our unsigned varint is a variant of unsigned [LEB128](https://en.wikipedia.org/wiki/LEB128) with some additional restrictions.

Specifically, unlike LEB128:

- Signed numbers are not supported.
- Leading zeros must be trimmed when encoding and must be rejected when decoding. The only number that can end in `0x0` is `0x0`.

### Spec

The encoding is:
- unsigned integers are serialized 7 bits at a time, starting with the least significant bits
- the most significant bit (msb) in each output byte indicates if there is a continuation byte (msb = 1)
- there are no signed integers
- integers are minimally encoded

Examples: 

```
1     => 00000001
127   => 01111111
128   => 10000000 00000001
255   => 11111111 00000001
300   => 10101100 00000010
16384 => 10000000 10000000 00000001
```

```
byte   # |              0 |            1   |          2     |
bit    # |c 6 5 4 3 2 1 0 |c 5 4 3 2 1 0 7 |c 4 3 2 1 0 7 6 |
         |----------------|----------------|----------------|
16384 => |1 0 0 0 0 0 0 0 |1 0 0 0 0 0 0 0 |0 0 0 0 0 0 0 1 |
```

Code that generates this.
```
package main

// test program. we can use the go one.
import (
  "encoding/binary" // varint is here
  "fmt"
)

func main() {
  ints := []uint64{1, 127, 128, 255, 300, 16384}
  for _, i := range ints {
    buf := make([]byte, 10)
    n := binary.PutUvarint(buf, uint64(i))

    fmt.Print(i, "\t=> ")
    for c := 0; c < n; c++ {
      fmt.Printf("%08b ", int(buf[c]))
    }
    fmt.Println()
  }
}
```



### Practical maximum of 9 bytes (for security)

For security, to avoid memory attacks, we use a "practical max" of 9 bytes. Though there is no theoretical limit, and future specs can grow this number if it is truly necessary to have code or length values equal to or larger than `2^63`.

For the forseeable future:

- Implementations MUST restrict the size of the varint to a max of 9 bytes (63 bits).
- A multiformat spec MAY explicitly declare a smaller maximum when using varints.
- A multiformat spec MAY NOT explicitly declare a larger maximum when using varints without first changing this spec.

### Incompatibilities with Go varints

This MSB-based unsigned varint is based on the [varint of the Go standard library](https://golang.org/src/encoding/binary/varint.go), which itself was based on the protocol buffers one.

However, we have two modifications:

- Multiformats varint only supports unsigned integers, the Go varint supports signed (using zig-zag encoding).
- Multiformats varints must be minimally encoded. That is, numbers must be encoded in the least number of bytes possible.

> What do we mean by minimally encoded?

Multiformat varints must be encoded in as few bytes as possible. To illustrate
the issue, take `{0x81 0x00}`. This is a valid golang varint encoding of 0x1.
However, the _minimal_ encoding of 0x1 is `{0x1}`.

## Implementations

* [Rust](https://github.com/paritytech/unsigned-varint)
* [JS](https://github.com/chrisdickinson/varint)
* [Go](https://github.com/multiformats/go-varint)

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/unsigned-varint/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

This repository is only for documents. All of these are licensed under the [CC-BY-SA 3.0](https://ipfs.io/ipfs/QmVreNvKsQmQZ83T86cWSjPu2vR3yZHGPm5jnxFuunEB9u) license © 2016 Protocol Labs Inc. Any code is under a [MIT](LICENSE) © 2016 Protocol Labs Inc.
