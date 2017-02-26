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

The MSB-based unsigned varint is based on the [varint of the Go standard library](https://golang.org/src/encoding/binary/varint.go) and [protocol buffers](), with some modifications:
- There is no max. The Go implementation has a "64-bit integer" max (using 10 bytes for representation). Thankfully, the Go implementation leaves the door open to grow the integer if need be. Here, we explicitly state there is no maxmimum. _There may always be a continuation bit._
- Implementations of each multiformat will give recommended maxes to avoid memory attacks, and varint implementations SHOULD either:
  - specify a soft max (a max in the implementation though not in the spec)
  - or require clients (like a `mulithash` implementation) to specify the max.

This format is simpler, and our varints are not expected to ever get beyond 32bits, as opposed to what you might find with group varints.

### Spec

The encoding is:
- unsigned integers are serialized 7 bits at a time, starting with the
  least significant bits
- the most significant bit (msb) in each output byte indicates if there
  is a continuation byte (msb = 1)

## Maintainers

Captain: [@jbenet](https://github.com/jbenet).

## Contribute

Contributions welcome. Please check out [the issues](https://github.com/multiformats/unsigned-varint/issues).

Check out our [contributing document](https://github.com/multiformats/multiformats/blob/master/contributing.md) for more information on how we work, and about contributing in general. Please be aware that all interactions related to multiformats are subject to the IPFS [Code of Conduct](https://github.com/ipfs/community/blob/master/code-of-conduct.md).

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

This repository is only for documents. All of these are licensed under the [CC-BY-SA 3.0](https://ipfs.io/ipfs/QmVreNvKsQmQZ83T86cWSjPu2vR3yZHGPm5jnxFuunEB9u) license © 2016 Protocol Labs Inc. Any code is under a [MIT](LICENSE) © 2016 Protocol Labs Inc.
