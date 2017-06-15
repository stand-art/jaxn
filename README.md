# Welcome to the home of JAXN

## Introduction

JAXN (pronounced "Jackson") is a standard for relaxed [JSON](https://tools.ietf.org/html/rfc7159). It targets humans writing JSON files manually (for example in configuration files), as well as providing a small set of extensions to improve the expressiveness and interoperability of JSON.

**JAXN IS CURRENTLY WORK-IN-PROGRESS**

Until we publish version 1.0 of JAXN, everything is work-in-progress. We appreciate ideas, feedback, criticsm and input from others, feel free to open an issue or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## Goals

* Add only a few, simple, human-readable and easy to understand extensions to JSON.
* Remain a strict sub-set of JSON. Valid JSON is valid JAXN.
* Agnostic towards the implementation language.
* Allow for simple, one-pass parsers.

## Extensions to JSON

* C-style comments:

  * `// single-line comment`.
  * `/* block comment */'.
  * Block-comments can not to nested.

* Unquoted C-style identifiers as object keys:

  * `{ foo: "Hello", bar: 42 }`.
  * TODO: Allow additional characters in identifiers? Candidates are `$`, `-` and `.`.

* Trailing comma in arrays and objects:

  * Allow `[1,2,3,]` and `{ foo: "Hello", bar: 42, }`.
  * But not `[,]`, `{,}`, `[,1,2]`, `[1,,2]`, etc.

* Strings:

  * Allow single-quote strings: `'This is a "single-quote" string. No really, it is!'`.
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
  * Just like the empty string, `$` itself is a valid, but empty, binary literal.
  * Allow optional dots for readability, e.g. `$49.6E.66.69.6E.69.74.79`.
  * Concatenation of binary string with each other and "normal" strings is allowed.

## Discussion

### White-space

Other libraries sometimes allow additional white-space characters. We do not see a real-world use-case for those, as we believe they are often added by mistake and we do not support people making mistakes, we support features that provide additional value.

### Unicode

Other libraries require Unicode support to implement certain extensions. While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places and additional burden on others, e.g. in the embedded world. JAXN does not require Unicode support on top of what is already required in JSON itself.

### String values

String values in JSON are required to be valid Unicode strings in order to be interoperable. JSON explains in RFC 7159 paragraph 8.2 why this is the case. JAXN specifies clearly how an implementation may threat those cases in order to increase interoperability. (TODO: Describe it!)

This extends the range of allowed values that a string can hold to basically a sequence of bytes where all byte values and combinations are allowed. This opens the door to handling of binary data, which is often required in real-world use-cases.

Some extensions like additional escape sequences from other libraries were dropped in favor of simplity and in light of the one, truly needed and best scaling extension: Raw strings. (TODO: Describe raw strings).

### Number values

NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 759. Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability. JAXN defines a fall-back path for libraries that can not handle those values internally, while encouraging library authors to accept NaN and Infinity as valid numeric values if possible.

### Binary strings

As JAXN allows for strings to contain arbirary data, a more consise and reasonable representation is required. As we target humans reading and writing JAXN files, we have choosen a hexadecimal representation over, say, Base64 as humans are notoriously bad at reading and writing Base64.

Implementations are encouraged to treat binary strings as a separate data type if possible. This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data. The latter is helpful when you are dumping data to, say, a log-file.

If an implementaiton supports a binary data type, the concatenation of binary data mixed with strings shall produce binary data.

## History

We are the authors of a JSON library, [taocpp/json](https://github.com/taocpp/json), and users were asking for relaxed JSON features. Naturally, we first checked for existing standards and the most promising candidate we found was [JSON5](http://json5.org). After some discussion, we figured that it doesn't quite fit the bill. It's a very good starting point, but in some cases we felt that JSON5 went too far, while in other cases it was limited by its strict ES5 ties.

Consequently, some of JAXN's extensions were inspired by, and are compatible with, JSON5. The following describes the differences between JSON5 and JAXN.

* Comments are the same.
* Handling of trailing commas is the same.
* Numbers are the same.

* JAXN does not allow additional white-space characters.
* JAXN does not allow multi-line strings (aka escaped new-lines within strings).
* JAXN does not allow escaping any character without special meaning.
* JAXN does not allow `$` in identifiers (but this might change...).

* JSON5 does not allow binary strings.
* JSON5 does not allow string concatenation (although string concatenation is available in ES5).
* JSON5 does not allow raw strings (TODO: work-in-progress/planned for JAXN).

With the exception of the last three bullet points, this means every valid JAXN string is also valid JSON5. We would like to extend interoperability, namely by these points:

* The goal should be that JAXN is, with the fewest exceptions possible, a sub-set of JSON5.
* Would JSON5 allow string concatenation? (It is part of ES5, right?)
* JSON5 currently does not allow raw strings. Can we come up with a common syntax?
* Binary strings are currently the biggest issue. Any ideas?
* We consider to extend identifiers to allow `$`, `-` and `.`. Are those valid for JSON5? The current JSON5 reference implementation does not allow `-` and `.`.

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
