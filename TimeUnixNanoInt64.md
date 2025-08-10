# [DRAFT] CBOR Tag for Nanoseconds Since Unix Epoch in Int64 Range

This document specifies a [CBOR](https://www.rfc-editor.org/info/std94)[1] tag for nanoseconds since Unix epoch as an integer in the range -2^63..2^63-1 inclusive.  This allows efficient and lossless encoding of nanoseconds that can represent time up to April 11, 2262.

```
Tag: ___
Data item: unsigned integer or negative integer
Semantics: Nanoseconds Since Unix Epoch in Int64 Range
Point of contact: Faye Amacker <faye.github@gmail.com>
Description of semantics: https://github.com/fxamacker/CBOR-Tag-Specs/blob/master/TimeUnixNanoInt64.md
```

## Semantics

Tag ___ can be applied to an unsigned integer (major type 0) or a negative integer (major type 1) to indicate the value represents nanoseconds since Unix Epoch (January 1, 1970 UTC).

The integer value representing nanoseconds must be limited to the range of a signed int64:
- Unsigned integer must be in the range 0..2^63-1 inclusive.
- Negative integer must be in the range -2^63..-1 inclusive.

## Rationale

Alternatives considered:
- Encoding nanoseconds to CBOR tag 0 with string can emit a ~30-byte RFC3339 string.
- Encoding nanoseconds to CBOR tag 1 with floating point is lossy and causes round-trip tests to fail.
- Supporting multiple precisions increases the cost of implementing, testing, and fuzzing codecs and user applications.
- Supporting a range greater than signed int64 increases edge cases to test and other issues in some programming languages.

Limiting the range of nanoseconds to fit into a signed int64 simplifies implementation, testing, and fuzzing for some programming languages.

The range of nanoseconds can represent time from Sept. 1677 to April 2262, which is more than enough for many use cases that need to serialize nanoseconds.

## Examples

Considerations in [Section 3.4.2 of RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html#section-3.4.2-4) (CBOR) for CBOR major type 1 are applicable to the tag defined by this document.  Notably, there is no universal standard for UTC count-of-seconds time before Unix Epoch so they may be interpreted by application requirements or by the standard library of the programming language used.

As one example, the Go (Golang) standard library can use negative or positive int64 nanoseconds to create Time objects that can be viewed as RFC3339 strings:

```
 -1<<63 nanoseconds is interpreted by Go as 1677-09-21T00:12:43.145224192Z
      0 nanoseconds is interpreted by Go as 1970-01-01T00:00:00Z
1<<63-1 nanoseconds is interpreted by Go as 2262-04-11T23:47:16.854775807Z
```

The CBOR codec[2] maintained by the author of this document can provide additional examples, test cases, and the reference implementation written in Go.

## References

[1] C. Bormann, and P. Hoffman. "Concise Binary Object Representation (CBOR)". [STD 94 / RFC 8949](https://www.rfc-editor.org/info/std94), December 2020.

[2] F. Amacker. "CBOR Codec in Go (Golang)". <https://github.com/fxamacker/cbor>.

## Author

Faye Amacker <faye.github@gmail.com>
