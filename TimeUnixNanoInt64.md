# [DRAFT2] CBOR Tag for Nanoseconds Since Unix Epoch for Int64

This document specifies a [CBOR](https://www.rfc-editor.org/info/std94)[1] tag for nanoseconds since Unix epoch as an integer in the range -2⁶³..2⁶³−1 inclusive.  This allows compact and lossless encoding of nanoseconds that can represent time up to April 11, 2262.

```
Tag: TBD
Data item: unsigned integer or negative integer
Semantics: Nanoseconds Since Unix Epoch for Int64
Point of contact: Faye Amacker <faye.github@gmail.com>
Description of semantics: https://github.com/fxamacker/CBOR-Tag-Specs/blob/master/TimeUnixNanoInt64.md
```

## Status

DRAFT2 (on Sunday, August 17, 2025) updated the title and text to improve clarity and added helpful suggestions from the IETF mailing list mentioning RFC 9581.

For example, DRAFT2 added the section "RFC 9581 Equivalence" with the equivalence provided by C. Bormann.

## Semantics

Tag TBD can be applied to an unsigned integer (major type 0) or a negative integer (major type 1) to indicate the value represents nanoseconds since Unix epoch (January 1, 1970 UTC).

The integer value representing nanoseconds must be in the range of -2⁶³..2⁶³−1 inclusive:
- Unsigned integer must be in the range 0..2⁶³−1 inclusive.
- Negative integer must be in the range -2⁶³..-1 inclusive.

Max nanoseconds (9223372036854775807) can represent the time "2262-04-11T23:47:16.854775807Z".

Min nanoseconds (-9223372036854775808) is interpreted as the time "1677-09-21T00:12:43.145224192Z" by the Go standard library.

There is no universal standard for UTC count-of-seconds time before Unix Epoch so negative values may be interpreted by application requirements or by the standard library of the programming language used.

Considerations in [Section 3.4.2 of RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html#section-3.4.2-4) (CBOR) for CBOR major type 1 are applicable to the tag defined by this document.

Although not required by this tag, generic CBOR codecs may want to provide a choice for users to allow or forbid negative values.

### RFC 9581 Equivalence

With tag number TBD and tag content nnn being an unsigned integer (in the range 0..2⁶³−1 inclusive) or a negative integer (in the range -2⁶³..-1 inclusive), the RFC 9581[1] equivalence is:

```
TBD(nnn) :=
1001({
  1: 0,    / 0 seconds elapsed since Unix epoch /
  -9: nnn  / nnn nanoseconds elapsed since Unix epoch in the range -2⁶³..2⁶³−1 inclusive /
})
```

For example, with tag number TBD and tag content being the unsigned integer 2⁶³−1, the RFC 9581 equivalence is:

```
TBD(9223372036854775807) :=
1001({
  1: 0,                    / 0 seconds elapsed since Unix epoch /
  -9: 9223372036854775807  / 9223372036854775807 nanoseconds elapsed since Unix epoch /
})
```

## Rationale

Currently, application protocol designers are able to use options specified by RFC 9581 for encoding time, duration, or period.  However, RFC 9581 was not intended to specify what must be implemented by generic CBOR codecs.

Generic CBOR codecs need a compact and lossless encoding of time with nanosecond precision, where:
- the total encoded size is competive with non-CBOR binary formats that emit ~10 bytes
- the allowed integer values must be in the range of int64_t (C99)
- the tag is optionless to support smaller code size and reduce complexity

Tags that don't require options simplify implementation, testing, fuzzing, etc. for generic CBOR codecs.

The int64_t range of nanoseconds can represent time from September 1677 to April 2262.  Applications that need a different time range are able to use CBOR tags defined by RFC 9581.

## Alternatives Considered

RFC 9581 was designed to provide a set of options to application protocol designers, and it was not intended to specify what must be implemented by generic CBOR codecs.

Additional alternatives considered include:
- Encoding nanoseconds to CBOR tag 0 with string can emit a ~30-byte RFC3339 string.
- Encoding nanoseconds to CBOR tag 1 with floating point is lossy and causes round-trip tests to fail.
- Adding options for precision would increase code size.
- Supporting a range greater than signed int64 increases edge cases to test and other issues for some programming languages.

## Reference Implementation

The CBOR codec[3] maintained by the author of this document can provide additional examples, test cases, and the reference implementation written in Go.

## References

[1] C. Bormann, and P. Hoffman. "Concise Binary Object Representation (CBOR)". [STD 94 / RFC 8949](https://www.rfc-editor.org/info/std94), December 2020.

[2] C. Bormann, B. Gamari, H. Birkholz. "Concise Binary Object Representation (CBOR) Tags for Time, Duration, and Period". [RFC 9581](https://www.rfc-editor.org/info/rfc9581), August 2024.

[3] F. Amacker. "CBOR Codec in Go (Golang)". <https://github.com/fxamacker/cbor>.

## Author

Faye Amacker <faye.github@gmail.com>
