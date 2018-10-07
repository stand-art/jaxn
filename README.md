# Welcome to JAXN

JAXN (pronounced "Jackson") is a standard that carefully extends [JSON](https://tools.ietf.org/html/rfc8259) with a few often-required additions to the data model, and with new syntax that makes it more human friendly.

> :exclamation: **JAXN IS CURRENTLY WORK-IN-PROGRESS** :exclamation:
>
> Until version 1.0 of JAXN is published, everything is considered work-in-progress, and anything might still change. Ideas, feedback and other input is welcome and appreciated. Please feel free to open an issue, or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## The JAXN Data Model

JAXN extends the JSON data model with the following points:

* Allows non-finite values `NaN`, `Infinity` and `-Infinity` for numbers.
* Adds a new primitive type for values representing binary data.

## The JAXN Text Representation

JAXN text representation extends the JSON text representation with the following points:

* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Binary Data](#binary-data)
* [Unquoted Object Keys](#unquoted-object-keys)
* [Trailing Comma](#trailing-comma)

#### Comments

* `# single-line comment`
* `// single-line comment`
* `/* block comment */`

#### Numbers

* Allow a leading `+` sign.
* Allow omission of leading or trailing zeros, e.g. `.5`, or `42.`.
* Add non-finite values `NaN` and `Infinity`.
* Add hexadecimal integer values, e.g. `0xDEADBEEF`.

#### Strings

* Add single-quoted strings, e.g. `'This is a "single-quote" string. No really, it is!'`.
* Add new escape sequences `\'`, `\v`, `\0` and `\u{X...}`.
* Add multiline strings with no escape sequences.
* Add concatenation of strings, e.g. `"Hello," + " world!"`.

#### Binary Data

* Binary data represents arbitrary byte sequences, not Unicode strings.
* Two syntactical variants that can be concatenated with each other.
* Hexdumped binary, e.g. `$48656c6c6f2c20776f726c6421`.
  * Allows optional dots, e.g. `$48.65.6c.6c.6f.2c.20.77.6f.72.6c.64.21`.
* Binary strings, e.g. `$"Hello, \x77orld!"`.
  * Only printable ASCII characters allowed, no control characters.
  * No `\uXXXX` or `\u{...}` escape sequences allowed, instead:
  * Add `\xXX` for arbitrary byte values.

#### Unquoted Object Keys

* Allow unquoted object keys, e.g. `{ foo: "Hello", bar: 42 }`.

#### Trailing Comma

* Allow `[1,2,3,]` and `{ foo: "Hello", bar: 42, }`.

## More information

* [Specification](Specification.md)
* [Discussion](Discussion.md)
* [ABNF grammar](jaxn.abnf)

## Libraries implementing JAXN

* [taocpp/json](https://github.com/taocpp/json)
* ...

Copyright (c) 2017-2018 Daniel Frey and Dr. Colin Hirsch
