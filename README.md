# proto3 Canonical Encoding Rules (CER)

This defines a set of encoding rules for Protocol Buffers 3 (proto3) for serializing messages deterministically
such that the serialized form is suitable for signing and encoding in cryptographic attestations (ex. Merkle trees).
Similar to [ASN.1](https://en.wikipedia.org/wiki/X.690#CER_encoding) and
[Cap'n Proto](https://capnproto.org/encoding.html#canonicalization), a set of "canonical encoding rules" (CER)
is used to define a canonical encoding where the basic proto3 specification does not do so. In this sense,
the default protocol buffers specification provides a set of "basic encoding rules" which are not deterministic,
and we extend that specification to support deterministic encoding for cryptographic use cases.

## Specification

### Fields Must be Serialized In Ascending Field Order

This is the most intuitive order in which to serialize fields.

### Default/Empty Values Must Not Be Serialized

Requiring default values to be serialized would prevent clients from an older version of a protocol from sending messages
to transaction processors which use a later version. Also, in proto3 there is no semantic distinction between empty and
default fields and thus serializing a default value is not intended to communicate any information. Thus the most canonical
behavior is to always omit fields with empty or default value from serialization.

### No Maps (for now)

While maps could have a canonical encoding, they are too problematic for cryptographically sensitive use cases and thus
excluded for now.

### Limitations

A recipient cannot determine if a message with unknown fields is canonical or not. Therefore all transaction processors which
receive messages with unknown fields should treat them as not canonical. In spite of this limitation, clients from an early
version of a protocol can send messages to transaction processors which understand a later version of the protocol without
causing a problem. Transaction processors would also reject messages intended for a later version of the protocol which they
do not understand which is likely the safest and most correct behavior in most cases.

## Implementations

*Please submit a [PR](https://github.com/regen-network/canonical-proto3/pulls) if you have implementation details to
add to this list.*

### [gogo protobuf](https://github.com/gogo/protobuf)

gogo proto mostly follows canonical encoding rules with the caveats listed below.

#### Don't use `gogoproto.nullable = false`

This causes default/empty fields to be emitted in binary and json encodings.

