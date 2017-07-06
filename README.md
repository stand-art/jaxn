# Welcome to JAXN

JAXN (pronounced "Jackson") is a standard that carefully extends [JSON](https://tools.ietf.org/html/rfc7159) with a few often-required additions to the data model and with new syntax that makes it more usable for humans.

> :exclamation: **JAXN IS CURRENTLY WORK-IN-PROGRESS** :exclamation:
>
> Until version 1.0 of JAXN is published, everything is considered work-in-progress, and anything might still change. Ideas, feedback and other input is welcome and appreciated. Please feel free to open an issue, or write to [`jaxn@icemx.net`](mailto:jaxn@icemx.net).

## The JAXN Data Model

JAXN extends the JSON data model with the following points:

* Allows non-finite values `NaN`, `Infinity` and `-Infinity` for numbers.
* Adds a new data type for binary data.
* Adds new data types for date/time/timestamps.

## The JAXN String Representation

JAXN string representation extends the JSON string representation with the following points:

* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Date / Time](#date--time)
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

* Allow single-quoted strings, e.g. `'This is a "single-quote" string. No really, it is!'`.
* Add new escape sequences `\'`, `\v`, `\0` and `\u{X...}`.
* Allow concatenation of strings like `"Hello," + " world!"`.
* TODO: Raw strings, i.e. strings that can span multiple lines without interpretation of escape sequences.

#### Date / Time

* Date: `2017-09-05`.
* Time: `10:23:54.345678`.
* Timestamps: `2017-09-05T10:23:54.345678`.
* Timestamps with timezone: `2017-09-05T10:23:54.345678+02:00`.

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

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
