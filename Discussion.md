# Discussion

## Data model extensions

Most "relaxed JSON" extensions focus on the syntax of the string representation. They sometimes do extend the data model, but they don't say so clearly or are even unaware of it. JAXN goes further, by clearly specifying which additional values and data types a library should support. This allows users to know what to expect from a JAXN-compatible library, or, looking at it from the other side, search for a JAXN-compatible library when they know that they need these extensions to the data model.

JAXN extends the JSON data model in two places.

1. Allow `NaN`, `Infinity` and `-Infinity` for numeric values.
2. Add a binary data type.

## White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake.

## Unicode

Other libraries require additional Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places an additional burden on others, e.g. in the embedded world. JAXN does not require additional Unicode support beyond what is already required by JSON itself.

## Comments

A library that conforms to JAXN **MUST** ignore comments, they shall not be part of the data model (unlike, say, XML libraries where XML comments are often nodes in the resulting object). JAXN comments, when parsed, shall not be exposed to the user of the library or in any way influence the behaviour of the library. This improves interoperability and ensures that the main concern why comments are not part of JSON is taken care of.

## Numeric values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. A JAXN-compatible library is required to accept NaN and Infinity as valid numeric values for their internal data model.

## String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. The JSON RFC 7159 explains in paragraph 8.2 why this is the case. JAXN does *not* change this. Unlike other libraries that allow escape sequences like `\xXX` for normal strings without specifying the semantics (properly), we keep the data model for strings intact.

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

## Binary strings

In real-world uses, one often needs to handle binary data. Representing this kind of data as strings requires, for example, hex- or base64-encoding. As JAXN recognizes the importance of binary data, we extend the data model of a JAXN-compatible library by an explicit binary type. For the representation in string form, we have chosen hex notation as base64 is human-unfriendly and adds additional implementation complexity. Having a binary type and a more direct representation allows for a more consise and reasonable representation.

Implementations must treat binary data as a separate data type. This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data. The latter is helpful when you are dumping data to, say, a log-file.

## Conversion to JSON

A JAXN data value may contain values that have no direct representation in JSON. Those are the non-finite numeric values and binary data. A library may chose to report an error when conversion to JSON string representation is requested. If may also chose to replace those values with strings. A JAXN-compatible library should use the following strings:

* `"NaN"` for a NaN. No other strings should be used, e.g. `"nan"`, `"+NaN"` or `"-NaN"`.
* `"Infinity"` and `"-Infinity"`. No other strings should be used, e.g. `"Inf"`, `"+Infinity"`, etc.
* Binary data should be represented as a string containing the hex encoded data, e.g. `"496E66696E697479"`.

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
