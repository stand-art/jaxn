# Specification

The following sections specify the syntax and semantics of the extensions that JAXN brings to JSON.

* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Date / Time](#date--time)
* [Binary Data](#binary-data)
* [Unquoted Object Keys](#unquoted-object-keys)
* [Trailing Comma](#trailing-comma)

Note: The grammar rules are an excerpt from the complete [JAXN grammar](jaxn.abnf).
The JAXN grammar is based on the JSON grammar given in [RFC 8259](https://tools.ietf.org/html/rfc8259), both are in ABNF syntax, defined in [RFC 5234](https://tools.ietf.org/html/rfc5234).

## Comments

#### Synopsis

Allows single-line and block comments.

#### Examples

* `# single-line comment`
* `// single-line comment`
* `/* block comment */`

#### Grammar

```abnf
comment = c-line / c-block

c-line = c-begin-line *( c-char )

c-begin-line = %x23 / %x2F.2F ; # or //

c-char = %x09 / %x20-10FFFF   ; Any HTAB or printable character

c-block = c-begin-block *( c-no-star / ( 1*c-star c-no-star-or-slash ) ) c-end-block

c-begin-block = c-slash c-star
c-end-block = 1*c-star c-slash

c-slash = %x2F                ; /
c-star = %x2A                 ; *

c-no-star = %x09 / %x0A / %x0D / %x20-29 / %x2B-10FFFF
c-no-star-or-slash = %x09 / %x0A / %x0D / %x20-29 / %x2B-2E / %x30-10FFFF

ws = *(
        %x20 /                ; Space
        %x09 /                ; Horizontal tab
        %x0A /                ; Line feed or New line
        %x0D /                ; Carriage return
	comment )             ; Comment
```

#### Semantics

Comments are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.

#### Notes

Single-line comments may not contain additional control characters.
A single-line comment ends at either the end of the line, or at the end of the input, whichever is encountered first.

Block comments do not nest.
In other words, occurrences of `/*` within a block comment are not interpreted as anything else other than part of the comment.

## Numbers

#### Synopsis

Allow non-finite values, hexadecimal notation of integer values, an optional leading plus sign, and relax the rules for redundant zeros.

#### Examples

* `42.`
* `+.5`
* `NaN`
* `Infinity`
* `-Infinity`
* `0xDEADBEEF`

#### Grammar

```abnf
number = [ plus / minus ] ( nan / inf / hex / dec )

nan = %x4E.61.4E              ; NaN

inf = %x49.6E.66.69.6E.69.74.79
                              ; Infinity

hex = zero x 1*HEXDIG         ; 0xXXX...

dec = ( int [ frac0 ] / frac1 ) [ exp ]

decimal-point = %x2E          ; .

digit1-9 = %x31-39            ; 1-9

e = %x65 / %x45               ; e E
x = %x78 / %x58               ; x X

exp = e [ plus / minus ] 1*DIGIT

frac0 = decimal-point *DIGIT
frac1 = decimal-point 1*DIGIT

int = zero / ( digit1-9 *DIGIT )

plus = %x2B                   ; +
minus = %x2D                  ; -
zero = %x30                   ; 0
```

#### Notes

JAXN adds non-finite values to the data model that can not be represented in JSON.
The spelling of the identifiers is case-sensitive.
JAXN allows `+NaN` and `-NaN` as alternatives for `NaN`, as well as `+Infinity` as an alternative to `Infinity`.

All other extensions are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.

## Strings

#### Synopsis

Allow single-quoted strings, additional escape sequences, multiline strings, and string concatenation.

#### Examples

* `"Add \0 or \v, even \' is allowed in a string."`
* `'That\'s right, you need to escape single-quotes in a single-quoted string.'`
* `'Oh, and \" is allowed even in a single-quote string.'`
* `"\u{1D11E} was my first love " + "and it will be my last."`
* `"""String with a \ and " characters - no escape sequences,`<br>`may contain line breaks"""`

#### Grammar

```abnf
string = string-part *( value-concat string-part )

string-part = m-d-string / m-s-string / d-string / s-string

m-d-string = 3d-quote *( m-d-char ) 3d-quote
m-s-string = 3s-quote *( m-s-char ) 3s-quote

m-d-char = *2d-quote ( %x09 / %x0A / %x0D / %x20-21 / %x23-10FFFF )
m-s-char = *2s-quote ( %x09 / %x0A / %x0D / %x20-26 / %x28-10FFFF )

d-string = d-quote *( s-char / s-quote ) d-quote
s-string = s-quote *( s-char / d-quote ) s-quote

s-char = unescaped /
         escape (
             %x22 /           ; "    double quote    U+0022
             %x27 /           ; '    single quote    U+0027
             %x5C /           ; \    reverse solidus U+005C
             %x2F /           ; /    solidus         U+002F
             %x30 /           ; 0    nul             U+0000
             %x62 /           ; b    backspace       U+0008
             %x66 /           ; f    form feed       U+000C
             %x6E /           ; n    line feed       U+000A
             %x72 /           ; r    carriage return U+000D
             %x74 /           ; t    tab             U+0009
             %x76 /           ; v    vtab            U+000B
             %x75 4HEXDIG /   ; uXXXX                U+XXXX
             %x75 %x7B 1*HEXDIG %x7D )
                              ; u{X...}              U+X...

escape = %x5C                 ; \
d-quote = %x22                ; "
s-quote = %x27                ; '

unescaped = %x20-21 / %x23-26 / %x28-5B / %x5D-10FFFF
```

#### Notes

* Each string (in a concatenation: individually) **MUST** be a sequence of Unicode characters.
* `\uXXXX` with UTF-16 surrogates **MUST** be handled before concatenation.
* `\u{X...}` **MUST NOT** encode surrogates.
* Multiline strings:
  * Are surrounded by three quotation marks (single or double quote) on each side and allow newline.
  * Are restricted to the source character set, horizontal tab is allowed.
  * Do not interpret escape sequences, a backslash `\` is just a literal backslash.
  * A newline immediately following the opening delimiter is trimmed.
  * All other characters remain intact.
* Concatenations can mix single- and double-quoted strings as well as single- or multiline-strings.
* Concatenation is a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
  It happens before the final string is passed on from the parser.

## Date / Time

#### Synopsis

Allows date, time, or timestamp values.
Four types need to be distinguished:

* A date (without a time or an offset). In the context of JAXN, this is referred to as a "local date".
* A time (without a date or an offset). In the context of JAXN, this is referred to as a "local time".
* A timestamp without an offset. In the context of JAXN, this is referred to as a "local date-time".
* A timestamp with an offset. In the context of JAXN, this is referred to as an "offset date-time".

#### Examples

* `2017-09-05`
* `10:23:54.345678`
* `2017-09-05 10:23:54.345678`
* `2017-09-05T10:23:54.345678`
* `2017-09-05 10:23:54.345678+02:00`
* `2017-09-05T10:23:54.345678+02:00`

#### Grammar

```abnf
time-value = local-date / local-time / local-date-time / offset-date-time

date-fullyear     = 4DIGIT
date-month        = 2DIGIT    ; 01-12
date-mday         = 2DIGIT    ; 01-28, 01-29, 01-30, 01-31 based on month/year

time-hour         = 2DIGIT    ; 00-23
time-minute       = 2DIGIT    ; 00-59
time-second       = 2DIGIT    ; 00-58, 00-59, 00-60 based on leap second rules
time-secfrac      = decimal-point 1*DIGIT

time-numoffset    = ( plus / minus ) time-hour ":" time-minute
time-offset       = "Z" / time-numoffset

local-time        = time-hour ":" time-minute ":" time-second [ time-secfrac ]
local-date        = date-fullyear "-" date-month "-" date-mday

local-date-time   = local-date ( "T" / " " ) local-time
offset-date-time  = local-date-time time-offset
```

#### Notes

The formats are described in [RFC 3339](https://tools.ietf.org/html/rfc3339).

The represented date, time or timestamp must be valid, e.g. not `2000-02-30`.

Each type is a unique type and shall not be confused with the others.
A round-trip must retain the type.

Space is allowed as a separator between date and time.
Two local date-time values are considered equal when both the date and the time part are equal, the separator is not part of the value.
Round trips may replace the separator for another, using capital `T` is recommended.

The precision of fractional seconds is implementation specific, but at least millisecond precision must be implemented.
Nanosecond precision is recommended.
If the value contains greater precision than the implementation can support, the additional fractional second digits must be truncated, not rounded.
Round trips may truncate fractional second digits, or add trailing zeros if necessary.

Two offset date-time values are considered equal when both the local date-time part as well as the offset are equal.
The values `2000-01-01T00:02:00+00:00` and `2000-01-01T00:00:00+02:00` are *not* considered equal in JAXN.
The offsets `+00:00`, `-00:00` and `Z` (and `z`) represent the same offset value, a round trip may interchange them.

Leap second support is optional, an implementation may choose not to support leap-seconds.
For interoperability, it is best *not* to use leap-seconds.

## Binary Data

#### Synopsis

Allow binary data as a separate type, in two forms: Hexdump or string.

#### Examples

* `$"Hello, \x77orld!"` (binary string)
* `$48656c6c6f2c20776f726c6421` (binary hex)
* `$48656c6c6f.2c20.776f726c64.21`
* `$48.65.6c.6c.6f.2c.20.77.6f.72.6c.64.21`

#### Grammar

```abnf
binary = b-value *( value-concat b-value )

b-value = dollar [ b-string / b-direct ]

b-string = b-s-string / b-d-string

b-d-string = d-quote *( b-char / s-quote ) d-quote
b-s-string = s-quote *( b-char / d-quote ) s-quote

b-char = b-unescaped /
         escape (
             %x22 /           ; "    double quote    0x22
             %x27 /           ; '    single quote    0x27
             %x5C /           ; \    reverse solidus 0x5C
             %x2F /           ; /    solidus         0x2F
             %x30 /           ; 0    nul             0x00
             %x62 /           ; b    backspace       0x08
             %x66 /           ; f    form feed       0x0C
             %x6E /           ; n    line feed       0x0A
             %x72 /           ; r    carriage return 0x0D
             %x74 /           ; t    tab             0x09
             %x76 /           ; v    vtab            0x0B
             %x78 2HEXDIG )   ; xXX                  0xXX

b-unescaped = %x20-21 / %x23-26 / %x28-5B / %x5D-7E

b-direct = b-part *( dot b-part )

b-part = 1*b-byte

b-byte = 2HEXDIG

dollar = %x24                 ; $
dot = %x2E                    ; .
```

#### Notes

* Binary data represents arbitrary byte sequences, not Unicode strings.
* In binary strings only printable ASCII characters are allowed, no control characters.
* No `\uXXXX` or `\u{...}` escape sequences allowed, instead:
* Escape sequence `\xXX` for arbitrary byte values.
* Concatenations can mix single- and double-quoted binary strings as well as hexdumped data.
* Concatenation is a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
  It happens before the final binary value is passed on from the parser.

## Unquoted Object Keys

#### Synopsis

Allow identifiers as unquoted object keys.

#### Example

`{ foo: "Hello", bar: 42 }`

#### Grammar

```abnf
member = key name-separator value

key = string / identifier

identifier = i-begin *i-continue

i-begin = ALPHA / %x24 / %x5F
i-continue = i-begin / DIGIT
```

#### Notes

Object keys are strings, wherefore the tokens `true`, `null`, and `false` are unambiguous shortcuts for `"true"`, `"null"`, and `"false"` when used in an object key position.
Strings in object key positions can use the extended syntax for strings including the single-quoted variant, additional escape sequences and string concatenation, however unquoted object keys can **not** be concatenated.

Unquoted object keys are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
They are passed on as normal strings from the parser.

## Trailing Comma

#### Synopsis

Allow trailing commas in arrays and objects.

#### Examples

* `[ 1, 2, 3, ]`
* `{ foo: "Hello", bar: 42, }`

#### Grammar

```abnf
value-sep-opt = [ value-separator ]

array = begin-array [ value *( value-separator value ) value-sep-opt ] end-array

object = begin-object [ member *( value-separator member ) value-sep-opt ] end-object
```

#### Semantics

The additional commas have no semantics.

#### Notes

The above grammar does not allow for adjacent commas (`[1,,2]`), a leading comma (`[,1]`), or placing a comma in an empty array or object (`[,]`).

Trailing commas are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.

Copyright (c) 2017-2018 Daniel Frey and Dr. Colin Hirsch
