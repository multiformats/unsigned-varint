# unsigned-varint

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-multiformats-blue.svg?style=flat-square)](https://github.com/multiformats/multiformats)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](https://webchat.freenode.net/?channels=%23ipfs)
[![](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> unsigned varint in use in multiformat specs

This unsigned varint (VARiable INTeger) format is for the use in all the multiformats.

There is time to discuss the details of this. All  multiformats are far from requiring this varint.

## Format

Our unsigned varint is an MSB based unsigned varint.

### Spec

The encoding is:
- unsigned integers are serialized 7 bits at a time, starting with the least significant bits
- the most significant bit (msb) in each output byte indicates if there is a continuation byte (msb = 1)
- there are no signed integers

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

For security, to avoid memory attacks, we use a "practical max" of 9 bytes. Though there is no theoretical limit, and future specs can grow this number if it is truly necessary to have code or length values larger than `2^63 - 1`. 

For the forseeable future:

- Implementations MUST restrict the size of the varint to a max of 9 bytes (63 bits).
- A multiformat spec MAY explicitly declare a smaller maximum when using varints.
- A multiformat spec MAY NOT explicitly declare a larger maximum when using varints without first changing this spec.

### Max differences from Go Varint

This MSB-based unsigned varint is based on the [varint of the Go standard library](https://golang.org/src/encoding/binary/varint.go), which itself was based on the protocol buffers one, and that one in turn was based on ...

However, we have two modifications:

- Multiformats varint only supports unsigned integers, the Go varint supports signed (using zig-zag encoding)
- Multiformats varint does not use the 9th byte's msb as part of the number. It never interprets 64-bit numbers from 9 bytes. The Go varint does do that.

> What is this about 9th byte msb in Go varint ...

In the Go implementation, the target is a "64-bit integer". Since the 64th bit bumps to 10 varint bytes, the authors chose to restrict the maximum size to 9-bytes and made the last byte's msb be part of the number. This means the Go implementation is incompatible with 128bit varints (protobuf), see the design note in [varint.go](https://golang.org/src/encoding/binary/varint.go). This also means growing the varint may be difficult or break things as numbers might then mean two different things.

Instead, in the multiformats unsigned-varint, we explicitly declare that our unsigned varints are _theoretically_ infinite, but _in practice_ limited to 9 bytes for security. This means:

- There may always be a continuation bit. 
- A continuation bit in the 9th byte MUST be ignored, and an error returned.
- Our unsigned ints are compatible with much larger integers (like 128-bit unsigned protobuf varints)
- Leaves door open for growing in the future if it is absolutely needed.
  - This gives us a large window of numbers (2^63 is a huge number), plenty big for these use cases.

This format is simpler, and our varints are not expected to ever get beyond 63bits, as opposed to what you might find with group varints.

## Maintainers

Captain: [@jbenet](https://github.com/jbenet).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/unsigned-varint/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

This repository is only for documents. All of these are licensed under the [CC-BY-SA 3.0](https://ipfs.io/ipfs/QmVreNvKsQmQZ83T86cWSjPu2vR3yZHGPm5jnxFuunEB9u) license © 2016 Protocol Labs Inc. Any code is under a [MIT](LICENSE) © 2016 Protocol Labs Inc.
