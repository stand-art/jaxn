# Welcome to JAXN

JAXN (pronounced "Jackson") is a standard for "relaxed [JSON](https://tools.ietf.org/html/rfc7159)". It targets humans writing JSON files manually (for example in configuration files), as well as providing a small set of extensions to improve the expressiveness and interoperability of JSON libraries.

## What is JAXN?

* A strict superset of JSON. Valid JSON is valid JAXN.
* Adds a few simple, human-readable and easy to understand extensions to JSON.
* Aims at wide-spread adoption from JSON libraries.
* Agnostic towards the implementation language.
* Allow for simple, one-pass parsers.

> **JAXN IS CURRENTLY WORK-IN-PROGRESS**
>
> Until we publish version 1.0 of JAXN, everything is considered work-in-progress. We appreciate ideas, feedback, criticism and input from others. Feel free to open an issue or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## Extensions to JSON

* C-style comments:

  * `// single-line comment`.
  * `/* block comment */`.
  * Block-comments can *not* be nested.

* Unquoted C-style identifiers as object keys:

  * `{ foo: "Hello", bar: 42 }`.
  * TODO: Allow additional characters in identifiers? Candidates are `$`, `-` and `.`.

* Trailing comma in arrays and objects:

  * Allow `[1,2,3,]` and `{ foo: "Hello", bar: 42, }`.
  * But not `[,]`, `{,}`, `[,1,2]`, `[1,,2]`, etc.

* Strings:

  * Allow single-quoted strings: `'This is a "single-quote" string. No really, it is!'`.
  * Additional escape sequences: `\'`, `\v`, `\0` and `\xXX`.
  * Allow concatenation of strings: `"Hello," + " world!"`.
  * TODO: Raw strings?

* Numbers:

  * Allow an explicit leading `+` sign.
  * Allow omitting leading or tailing zeroes, e.g. `.5`, or `42.` are valid numbers.
  * Add special values `NaN` and `Infinity`.
  * Add hexadecimal integer values, e.g. `0xDEADBEEF`.

* Binary strings:

  * Instead of escape sequences in a string, allow binary data directly, e.g. `$496E66696E697479`.
  * Just like the empty string, `$` itself is a valid, but empty, binary string.
  * Allow optional dots for readability, e.g. `$49.6E.66.69.6E.69.74.79`.
  * Concatenation of binary strings with each other and "normal" strings is allowed.

## Discussion

### Comments

A library that conforms to JAXN *must* ignore comments, they shall not be part of the data model (unlike, say, XML libraries where XML comments are often nodes in the resulting object). JAXN comments, when parsed, shall not be exposed to the user of the library or in any way influence the behaviour of the library. This improves interoperability and ensures that the main concern why comments are not part of JSON is taken care of.

### White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake and we do not support people making mistakes, we support features that provide additional value.

### Data model extensions

Most "relaxed JSON" extensions focus on the syntax of the string representation. They sometimes do extend the data model, but they don't say so clearly. JAXN goes further, by clearly specifying which additional values and data types a library could support and how such support should be implemented. This allows users to know what to expect from a JAXN-compatible library, or, looking at it from the other side, search for a JAXN-compatible library when they know that they need certain extensions to the data model.

JAXN extends the JSON data model by three points:

* Additional non-finite values for numeric values, i.e. `NaN`, `Infinity` and `-Infinity`.
* Allow strings to contain arbitrary byte sequences.
* Add a binary data type.

### Unicode

Other libraries require Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places and additional burden on others, e.g. in the embedded world. JAXN does not require Unicode support on top of what is already required in JSON itself.

### String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. The JSON RFC 7159 explains in paragraph 8.2 why this is the case.

JAXN extends the range of allowed values that a string can hold to basically any sequence of bytes where all byte values and combinations are allowed. This opens the door to handling of binary data, which is often required in real-world use-cases. This extension of the data model is a requirement for all JAXN-compatible libraries.

Some extensions like additional escape sequences from other libraries were dropped in favor of simplity and in light of the one, truly needed and best scaling extension: Raw strings. (TODO: Describe raw strings).

### Number values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. JAXN defines a fall-back path for libraries that can not handle those values internally, while encouraging library authors to accept NaN and Infinity as valid numeric values if possible.

### Binary strings

As JAXN allows for strings to contain arbirary data, a more consise and reasonable representation is required. As we target humans reading and writing JAXN files, we have choosen a hexadecimal representation over, say, Base64 as humans are notoriously bad at reading and writing Base64.

Implementations are encouraged to treat binary strings as a separate data type if possible. This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data. The latter is helpful when you are dumping data to, say, a log-file.

If an implementaiton supports a binary data type, the concatenation of binary data mixed with strings produces binary data.

## Libraries implementing JAXN

* [taocpp/json](https://github.com/taocpp/json)
* ...

## History

As authors of a JSON library, users were asking for relaxed JSON features. Naturally, we first checked for existing standards and the most promising candidate we found was [JSON5](http://json5.org). After some discussion, we figured that it doesn't quite fit the bill. It's a very good starting point, but in some cases we felt that JSON5 went too far, while in other cases it was limited by its strict ES5 ties.

Consequently, some of JAXN's extensions were inspired by, and are compatible with, JSON5. The following describes the differences between JSON5 and JAXN.

* Comments are the same.
* Handling of trailing commas is the same.
* Numbers are the same.
* JAXN does not allow:

  * Additional white-space characters.
  * `$` in identifiers (but this might change...).
  * Escaping any character without special meaning.
  * Multi-line strings (aka escaped new-lines within strings).

* JSON5 does not allow:

  * String concatenation (although string concatenation is available in ES5).
  * Raw strings (TODO: work-in-progress/planned for JAXN).
  * Binary strings.

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
