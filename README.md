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

## JSON

In addition to the rules below, signable canonical protobuf JSON must follow https://gibson042.github.io/canonicaljson-spec/.

### No default/empty values

Remove all fields whose value is `0`, `false`, `""`, `null`, `[]`, or `{}`.

### `Timestamp` and `Duration` should use 9 fractional digits

The [proto3 JSON specification](https://developers.google.com/protocol-buffers/docs/proto3#json) states that these types
can use 0, 3, 6 or 9 digits in JSON output. For a simple deterministic encoding, we specificy the most precise of these
9 digits.

## Implementations

*Please submit a [PR](https://github.com/regen-network/canonical-proto3/pulls) if you have implementation details to
add to this list.*

Implementations should specify one of the following levels of alignment:
* Level 1: there are clear rules to follow in order to make this implementation follow CER
* Level 2: this implementation has explicity code generation flags or static linting tools for safely supporting CER
* Level 3: this implementation provides a zero-allocation "is_canonical" or "unmarshal_canonical" method for checking
if a message is canonical

Note that level 1 and 2 implementations can still verify that a message is canonical by re-encoding it canonically and comparing.

### [gogo protobuf](https://github.com/gogo/protobuf) - Level 1

gogo proto mostly follows canonical encoding rules with the caveats listed below.

#### Don't use `gogoproto.nullable = false`

This causes default/empty fields to be emitted in binary and json encodings.

