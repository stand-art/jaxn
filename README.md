# Welcome to JAXN

JAXN (pronounced "Jackson") is a "relaxed JSON", a standard that carefully extends [JSON](https://tools.ietf.org/html/rfc7159) with new syntax that makes it more tractable for humans, and with some often-required features that make the data model more powerful.

> **JAXN IS CURRENTLY WORK-IN-PROGRESS**
>
> Until version 1.0 of JAXN is published, everything is considered work-in-progress, and anything might still change. Ideas, feedback and other input is welcome and appreciated. Please feel free to open an issue, or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## What is JAXN?

* A strict superset of JSON, every valid JSON document is also JAXN.
* A syntactic extension to JSON that makes it more tractable for humans.
* An extended JSON that adds often required features to the data model.
* An extended JSON that improves the interoperability between libraries.
* Simple to parse in a single pass without (much) look-ahead.
* An implementation-language agnostic standard.

## Extensions to JSON

#### C and C++-Style Comments

* `// single-line comment`
* `/* block comment */`
  * Block-comments can *not* be nested.

#### Trailing Comma in Arrays and Objects

* Allow `[1,2,3,]` and `{ foo: "Hello", bar: 42, }`.
  * **Not** `[,]`, `{,}`, `[,1,2]`, `[1,,2]`, or similar.

#### Numbers

* Allow leading `+` sign.
* Allow omission of leading or tailing zeroes.
  * E.g. `.5`, or `42.` are valid JAXN numbers.
* Add special values `NaN` and `Infinity`.
* Add hexadecimal integer values, e.g. `0xDEADBEEF`.

#### Strings

* Allow single-quoted strings, e.g. `'This is a "single-quote" string. No really, it is!'`.
* Add new escape sequences `\'`, `\v`, `\0` and `\u{X...}`.
  * `\u{X...}` is not allowed to encode UTF-16 code units corresponding to surrogates.
* Allow concatenation of strings like `"Hello," + " world!"`.
  * Each string (in a concatenation: individually) MUST be a sequence of Unicode characters.
  * `\uXXXX` with UTF-16 surrogates MUST be handled before concatenation.
* TODO: Raw strings, i.e. strings that can span multiple lines without interpretation of escape sequences.

#### Object Keys

* Unquoted C-style identifiers are allowed as object keys.
  * For example `{ foo: "Hello", bar: 42 }`.
  * TODO: Allow `$`, `-` and/or `.` in identifiers? Others?

#### Binary Data

* Binary data represents arbitrary byte sequences, not Unicode strings.
* Two syntactical variants that can be concatenated with each other.
* Binary strings, e.g. `$"Hello, world!"`.
  * Single- or double-quoted.
  * Only printable ASCII characters allowed, no control characters.
  * No `\uXXXX` or `\u{...}` escape sequences allowed, instead:
  * Add `\xXX` for arbitrary byte values.
* Hexdumped binary, e.g. `$496E66696E697479`.
  * Allows optional dots, e.g. `$49.6E.66.69.6E.69.74.79`.
    * Arbitrary mixed lengths of even-length sub-groups, e.g. `$30.020101.020101`.
* The binary prefix `$` by itself is valid and represents an empty binary data.
* Concatenation of regular strings and binary strings or binary data is not allowed.

## Discussion

#### Comments

A library that conforms to JAXN *must* ignore comments, they shall not be part of the data model (unlike, say, XML libraries where XML comments are often nodes in the resulting object). JAXN comments, when parsed, shall not be exposed to the user of the library or in any way influence the behaviour of the library. This improves interoperability and ensures that the main concern why comments are not part of JSON is taken care of.

#### White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake and we do not support people making mistakes, we support features that provide additional value.

#### Data model extensions

Most "relaxed JSON" extensions focus on the syntax of the string representation. They sometimes do extend the data model, but they don't say so clearly or are even unaware of it. JAXN goes further, by clearly specifying which additional values and data types a library could support and how such support should be implemented. This allows users to know what to expect from a JAXN-compatible library, or, looking at it from the other side, search for a JAXN-compatible library when they know that they need certain extensions to the data model.

JAXN extends the JSON data model in two places.

1. Allow `NaN`, `Infinity` and `-Infinity` for numeric values.
2. Add a binary data type.

#### Numeric values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. A JAXN-compatible library is required to accept NaN and Infinity as valid numeric values for their internal data model.

#### Unicode

Other libraries require Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places and additional burden on others, e.g. in the embedded world. JAXN does not require Unicode support beyond what is already required by JSON itself.

#### String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. The JSON RFC 7159 explains in paragraph 8.2 why this is the case. JAXN does *not* change this. Unlike other libraries that allow escape sequences like `\xXX` for normal strings without specifying the semantics, we keep the data model for strings intact.

More precisely, in the context of JAXN we need to distinguish four "levels" of a string value.

1. The representation string as encoded in the JAXN text.
2. The representation string as sequence of Unicode characters.
3. The represented string as sequence of Unicode characters.
4. The represented string as encoded depending on the implementation.

Regarding 1, JAXN allows only UTF-8, or also UTF-16 and UTF-32 encoding?

> (JSON allows UTF-8, UTF-16 and UTF-32, forbids writing byte-order markers, but encourages parsers to ignore them.)

Regarding the step from 1 to 2, a JAXN text MUST be a sequence of Unicode code points and any encoding error encountered during parsing is a fatal error.

> (RFC 7159 does not mention encoding errors in a JSON text?)

Regarding the step from 2 to 3, the sequence of represented Unicode code points is obtained from the sequence of representation code points by replacing escape sequences with the escaped code points (or, in the case of UTF-16 surrogates, temporary code units), and by merging the code units of subsequent high and low UTF-16 surrogates into a single code point.

> (RFC 7159 specifies how to encode code points not in the BMP with a 12-character encoding consisting of two `\uXXXX` escape sequences using UTF-16 surrogate pairs, but does not mandate a specific behaviour when the merging of surrogates fails, noting only that it could be "unpredictable" including "fatal".)

There appears to be an implicit silent consensus among JSON libraries to ignore unpaired surrogates. For now the JAXN standard allows unpaired surrogates. Therefore, at level 3, not all represented strings are actually valid sequences of unicode characters. (If the implementation uses UTF-8 as encoding, the final string encoding on level 4 can contain UTF-8 sequences for UTF-16 surrogates.) A JAXN implementation MAY choose to forbid unpaired surrogates, a behaviour compatible with RFC 7159.

JAXN string concatentation MUST be performed at level 3 or 4.

This implies that the merging of surrogate pairs, and the decision of whether a string contains unpaired surrogates, MUST be performed before concatentation.

Like JSON, JAXN standard does not talk about level 4, an implementation can store and encode a sequence of Unicode code points in any way it likes.

#### Binary strings

In real-world uses, one often needs to handle binary data. Representing this kind of data as strings requires, for example, hex- or base64-encoding. As JAXN recognizes the importance of binary data, we extend the data model of a JAXN-compatible library by an explicit binary type. For the representation in string form, we have chosen hex notation as base64 is human-unfriendly and adds additional implementation complexity. Having a binary type and a more direct representation allows for a more consise and reasonable representation.

Implementations must treat binary data as a separate data type. This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data. The latter is helpful when you are dumping data to, say, a log-file.

#### Conversion to JSON

A JAXN data value may contain values that have no direct representation in JSON. Those are the non-finite numeric values and binary data. A library may chose to report an error when conversion to JSON string representation is requested. If may also chose to replace those values with strings. A JAXN-compatible library should use the following strings:

* `"NaN"` for a NaN. No other strings should be used, e.g. `"nan"`, `"+NaN"` or `"-NaN"`.
* `"Infinity"` and `"-Infinity"`. No other strings should be used, e.g. `"Inf"`, `"+Infinity"`, etc.
* Binary data should be represented as a string containing the hex encoded data, e.g. `"496E66696E697479"`.

## Libraries implementing JAXN

* [taocpp/json](https://github.com/taocpp/json)
* ...

## JAXN Test Suite

> **Please note that this test suite does not yet exist**, this section is to discuss its development...

The JAXN test suite is intended to cover all aspects and details of the JAXN encoding.
A library that passes all tests can be considered JAXN compliant.
The test suite contains different categories of testcases, each consisting of one or more data files.

For the purpose of this document, two JSON files, or two JAXN files, are considered equivalent if they describe the same data.
This can be checked by first parsing and then printing the two files with *TBD tools in taocpp/json* and checking whether the outputs are equal.

#### Invalid JAXN

Each testcase consists of one input file that does not conform to the JAXN standard.
Reading such a file must generate an error.

#### JAXN encoded JSON

Each testcase consists of one valid JAXN input file, and one valid JSON reference file.
The JAXN input files in this category remain within the JSON data model.
The JAXN input file must be parsed and then printed to a JSON output file.
The test passes if the JSON output file and the JSON reference file are equivalent.

#### JAXN beyond JSON

Each testcase consists of one valid JAXN input file, and one valid JAXN reference file.
The JAXN input file must be parsed and then printed to a JAXN output file.
The test passes if the JAXN output file and the JAXN reference file are equivalent.

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
