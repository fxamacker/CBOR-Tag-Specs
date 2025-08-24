# CBOR Tag for Nanoseconds Since Unix Epoch UTC for Int64

This document specifies a [CBOR](https://www.rfc-editor.org/info/std94)[1] tag for nanoseconds since Unix Epoch UTC as an integer in the range -2⁶³..2⁶³−1 inclusive.  This allows compact and lossless encoding of nanoseconds that can represent POSIX time[2] up to April 11, 2262.

```
Tag: TBD
Data item: unsigned integer or negative integer
Semantics: Nanoseconds Since Unix Epoch UTC for Int64
Point of contact: Faye Amacker <faye.github@gmail.com>
Description of semantics: https://github.com/fxamacker/CBOR-Tag-Specs/blob/master/TimeUnixNanoInt64.md
```

## Semantics

Tag TBD can be applied to an unsigned integer (major type 0) or a negative integer (major type 1) to indicate the value represents nanoseconds elapsed since 1970-01-01T00:00Z UTC.

The integer value representing nanoseconds must be in the range of -2⁶³..2⁶³−1 inclusive:
- Unsigned integer must be in the range 0..2⁶³−1 inclusive.
- Negative integer must be in the range -2⁶³..-1 inclusive.

The maximum nanoseconds elapsed (2⁶³−1) represents the time 2262-04-11T23:47:16.854775807Z UTC.

Equivalent seconds elapsed since the epoch are interpreted according to RFC 8949 for CBOR tag 1, so:
- Nonnegative seconds are interpreted according to POSIX time for seconds elapsed since the epoch.
- Negative seconds may be interpreted according to application requirements because it isn't defined by POSIX time. See "Negative Integer Values" section below for more details.

Application protocols should specify whether or not negative values are allowed.  Generic CBOR codecs may want to provide a choice for users to allow or forbid negative values.  

### RFC 9581 Equivalence

With tag number TBD and tag content nnn being an unsigned integer (in the range 0..2⁶³−1 inclusive) or a negative integer (in the range -2⁶³..-1 inclusive), the RFC 9581[3] equivalence is:

```
TBD(nnn) :=
1001({
  1: 0,    / 0 seconds elapsed since the epoch 1970-01-01T00:00Z UTC /
  -9: nnn  / nnn nanoseconds elapsed since the epoch in the range -2⁶³..2⁶³−1 inclusive /
})
```

For example, with tag number TBD and tag content being the unsigned integer 2⁶³−1, the RFC 9581 equivalence is:

```
TBD(9223372036854775807) :=
1001({
  1: 0,                    / 0 seconds elapsed since the epoch 1970-01-01T00:00Z UTC /
  -9: 9223372036854775807  / 9223372036854775807 nanoseconds elapsed since the epoch /
})
```

### Nonnegative Integer Values

Nonnegative integer values represent nanoseconds elapsed on or after the epoch 1970-01-01T00:00Z UTC.

Like CBOR tag 1, nonnegative seconds elapsed since the epoch are interpreted according to POSIX time.  POSIX time ignores leap seconds and every day represents exactly 86400 seconds, so there is a discontinuity of 1-second several times per decade.  

### Negative Integer Values

As noted by [Section 3.4.2 of RFC 8949](https://www.rfc-editor.org/rfc/rfc8949.html#section-3.4.2-4) (CBOR), there is no universal standard for UTC count-of-seconds time before January 1, 1970 UTC, so negative values may be interpreted by application requirements.  When application requirements are not defined, negative values may be interpreted by the standard library of the programming language used.

As just one example, a generic CBOR codec implemented in Go may interpret the minimum nanoseconds (-2⁶³) as the time 1677-09-21T00:12:43.145224192Z, since that is the interpretation provided by the Go standard library.

## Rationale (remove this section in final version)

Users of a generic CBOR codec[4] have been requesting a compact and lossless encoding of nanosecond precision time in UTC.  And they want the encoding to be compatible between different generic CBOR codecs on different platforms.  With some exceptions, users prefer not to use the codec's extension API to integrate their own handling of CBOR tags.

In addition, third-party projects evaluate binary serialization formats by using generic codecs to compare serialization speed, memory use, and encoded size.

When serializing nanosecond precision time values, generic CBOR codecs are currently at a disadvantage.  For example, a non-CBOR codec uses a 10-byte encoding (2-byte prefix 0xd7ff and 8-byte value being an unsigned 34-bit seconds and 30-bit nanoseconds).

Applications and generic CBOR codecs need a compact and lossless encoding of time with nanosecond precision, where:
- the total encoded size is competive with non-CBOR binary formats that emit ~10 bytes
- the allowed integer values must be in the range of int64_t (C99)
- the tag supports small code size and fast speed

The int64_t range of nanoseconds can represent time from September 1677 to April 2262.  Applications that need a different time range are able to use CBOR tags defined by RFC 9581.

## Alternatives Considered (remove this section in final version)

RFC 9581 was designed to provide a set of options to application protocol designers, and it was not intended to specify what must be implemented by generic CBOR codecs.

Additional alternatives considered include:
- Encoding nanoseconds to CBOR tag 0 with string can emit a ~30-byte RFC3339 string.
- Encoding nanoseconds to CBOR tag 1 with floating point is lossy and causes round-trip tests to fail.
- Supporting multiple precisions (instead of only nanoseconds) would increase code size and other precisions are already supported by existing CBOR tags.
- Supporting a range that exceeds int64_t creates more edge cases to test and other issues for some programming languages.

## References

[1] C. Bormann, and P. Hoffman. "Concise Binary Object Representation (CBOR)". [STD 94 / RFC 8949](https://www.rfc-editor.org/info/std94), December 2020.

[2] IEEE, "The Open Group Base Specifications Issue 7", Section 4.16 Seconds Since the Epoch, IEEE Std 1003.1-2017, 2018, <http://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap04.html#tag_04_16>. 

[3] C. Bormann, B. Gamari, H. Birkholz. "Concise Binary Object Representation (CBOR) Tags for Time, Duration, and Period". [RFC 9581](https://www.rfc-editor.org/info/rfc9581), August 2024.

[4] F. Amacker. "CBOR Codec in Go (Golang)". https://github.com/fxamacker/cbor.

## Author

Faye Amacker <faye.github@gmail.com>
