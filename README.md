# Welcome to the home of JAXN

## Introduction

JAXN (pronounced "Jackson") is a standard for "relaxed [JSON](https://tools.ietf.org/html/rfc7159)". It targets humans writing JSON files manually (for example in configuration files), as well as providing a small set of extensions to improve the expressiveness and interoperability of JSON.

**JAXN IS CURRENTLY WORK-IN-PROGRESS**

Until we publish version 1.0 of JAXN, everything is considered work-in-progress. We appreciate ideas, feedback, criticsm and input from others. Feel free to open an issue or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## Goals

* JAXN adds only a few, simple, human-readable and easy to understand extensions to JSON.
* JAXN is a strict superset of JSON. Valid JSON is valid JAXN.
* JAXN is agnostic towards the implementation language.
* JAXN allow for simple, one-pass parsers.

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

A library that conforms to JAXN *must* ignore comments, they shall not be part of the data model (unlike, say, XML libraries where XML comments are often nodes in the resulting object). JAXN comments, when parsed, shall not be exposed to the user of the library or in any way influence the behaviour of the library. This ensures that the main concern why comments are not part of JSON is guaranteed for a JAXN-compatible library.

### White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake and we do not support people making mistakes, we support features that provide additional value.

### Unicode

Other libraries require Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places and additional burden on others, e.g. in the embedded world. JAXN does not require Unicode support on top of what is already required in JSON itself.

### String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. JSON explains in RFC 7159 paragraph 8.2 why this is the case. JAXN specifies clearly how an implementation may threat those cases in order to increase interoperability. (TODO: Describe it!)

This extends the range of allowed values that a string can hold to basically a sequence of bytes where all byte values and combinations are allowed. This opens the door to handling of binary data, which is often required in real-world use-cases.

Some extensions like additional escape sequences from other libraries were dropped in favor of simplity and in light of the one, truly needed and best scaling extension: Raw strings. (TODO: Describe raw strings).

### Number values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. JAXN defines a fall-back path for libraries that can not handle those values internally, while encouraging library authors to accept NaN and Infinity as valid numeric values if possible.

If possible, a JAXN-compatible library shall store those values as such. If, for some reason, the library/platform does not support IEEE 754, all non-finite values *must* be stored as one of three string values: "NaN", "Infinity", or "-Infinity", no other representation ("-NaN", "nan", "+Infinity", "Inf", etc.) is allowed. This rule improves interoperability between libraries. The user of the library is still free to replace values as appropriate for the business-logic.

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
