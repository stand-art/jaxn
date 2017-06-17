# Welcome to JAXN

JAXN (pronounced "Jackson") is a standard for "relaxed [JSON](https://tools.ietf.org/html/rfc7159)". It targets humans writing JSON files manually (for example in configuration files), as well as providing a small set of extensions to improve the expressiveness and interoperability of JSON libraries.

> **JAXN IS CURRENTLY WORK-IN-PROGRESS**
>
> Until we publish version 1.0 of JAXN, everything is considered work-in-progress. We appreciate ideas, feedback, criticism and input from others. Feel free to open an issue or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## What is JAXN?

* A strict superset of JSON. Valid JSON is valid JAXN.
* Adds a few simple, human-readable and easy to understand extensions to JSON.
* Aims at wide-spread adoption from JSON libraries.
* Agnostic towards the implementation language.
* Allow for simple, one-pass parsers.

## Extensions to JSON

* C-style comments:

  * `// single-line comment`.
  * `/* block comment */`.
  * Block-comments can *not* be nested.

* Trailing comma in arrays and objects:

  * Allow `[1,2,3,]` and `{ foo: "Hello", bar: 42, }`.
  * But not `[,]`, `{,}`, `[,1,2]`, `[1,,2]`, etc.

* Numbers:

  * Allow an explicit leading `+` sign.
  * Allow omitting leading or tailing zeroes, e.g. `.5`, or `42.` are valid numbers.
  * Add special values `NaN` and `Infinity`.
  * Add hexadecimal integer values, e.g. `0xDEADBEEF`.

* Strings:

  * Allow single-quoted strings: `'This is a "single-quote" string. No really, it is!'`.
  * Additional escape sequences: `\'`, `\v`, `\0` and `\u{...}`.
  * Allow concatenation of strings: `"Hello," + " world!"`.
  * Each string fragment must be valid Unicode, i.e. surrogate pairs *must not* be splitted.
  * TODO: Raw strings?

* Unquoted C-style identifiers as object keys:

  * `{ foo: "Hello", bar: 42 }`.
  * TODO: Allow additional characters in identifiers? Candidates are `$`, `-` and `.`.

* Binary strings:

  * Support for binary strings is optional!
  * Binary strings are byte sequences, not Unicode strings.
  * Prefixed by a dollar sign.
  * Quoted binary strings, e.g. `$"Hello, world!"`.
    * Single- or double-quoted.
    * Only printable ASCII characters, no control characters.
    * No `\uXXXX` or `\u{...}` escape sequences for Unicode code points.
    * Add `\xXX` for escaped bytes.
  * Direct binary strings, e.g. `$496E66696E697479`.
    * Allow optional dots, e.g. `$49.6E.66.69.6E.69.74.79`.
  * Just like the empty string, `$` itself is a valid, but empty, binary string.
  * Allow concatenation of binary strings.
  * Concatenation of normal and binary strings is not allowed.

## Discussion

### Comments

A library that conforms to JAXN *must* ignore comments, they shall not be part of the data model (unlike, say, XML libraries where XML comments are often nodes in the resulting object). JAXN comments, when parsed, shall not be exposed to the user of the library or in any way influence the behaviour of the library. This improves interoperability and ensures that the main concern why comments are not part of JSON is taken care of.

### White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake and we do not support people making mistakes, we support features that provide additional value.

### Data model extensions

Most "relaxed JSON" extensions focus on the syntax of the string representation. They sometimes do extend the data model, but they don't say so clearly or are even unaware of it. JAXN goes further, by clearly specifying which additional values and data types a library could support and how such support should be implemented. This allows users to know what to expect from a JAXN-compatible library, or, looking at it from the other side, search for a JAXN-compatible library when they know that they need certain extensions to the data model.

JAXN extends the JSON data model by two points:

* Additional non-finite values for numeric values, i.e. `NaN`, `Infinity` and `-Infinity`.
* Add a binary data type.

### Numeric values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. A JAXN-compatible library is required to accept NaN and Infinity as valid numeric values for their internal data model.

### Unicode

Other libraries require Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places and additional burden on others, e.g. in the embedded world. JAXN does not require Unicode support on top of what is already required in JSON itself.

### String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. The JSON RFC 7159 explains in paragraph 8.2 why this is the case. JAXN does *not* change that. Unlike other libraries that allow escape sequences like `\xXX` for normal strings without specifying the semantics, we keep the data model for strings intact.

### Binary strings

In real-world uses, one often needs to handle binary data. Representing this kind of data as strings requires, for example, hex- or base64-encoding. As JAXN recognizes the importance of binary data, we extend the data model of a JAXN-compatible library by an explicit binary type. For the representation in string form, we have chosen hex notation as base64 is human-unfriendly and adds additional implementation complexity. Having a binary type and a more direct representation allows for a more consise and reasonable representation.

Implementations must treat binary data as a separate data type. This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data. The latter is helpful when you are dumping data to, say, a log-file.

### Conversion to JSON

A JAXN data value may contain values that have no direct representation in JSON. Those are the non-finite numeric values and binary data. A library may chose to report an error when conversion to JSON string representation is requested. If may also chose to replace those values with strings. A JAXN-compatible library should use the following strings:

* `"NaN"` for a NaN. No other strings should be used, e.g. `"nan"`, `"+NaN"` or `"-NaN"`.
* `"Infinity"` and `"-Infinity"`. No other strings should be used, e.g. `"Inf"`, `"+Infinity"`, etc.
* Binary data should be represented as a string containing the hex encoded data, e.g. `"496E66696E697479"`.

## Libraries implementing JAXN

* [taocpp/json](https://github.com/taocpp/json)
* ...

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
