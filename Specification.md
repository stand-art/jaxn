# Specification

This document is the normative specification of JAXN.

JAXN is a data representation and interchange format based on JSON.

Only the differences between JAXN and JSON are specified.

JSON is to be understood as the version defined in [RFC 8259](https://tools.ietf.org/html/rfc8259).

* [Restrictions](#restrictions)
* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Binary Data](#binary-data)
* [Unquoted Names in Objects](#unquoted-names-in-objects)
* [Trailing Comma](#trailing-comma)

Note: The grammar rules given below are excerpts from the complete [JAXN grammar](jaxn.abnf).
The JAXN grammar is based on the JSON grammar, and both are in ABNF syntax as defined in [RFC 5234](https://tools.ietf.org/html/rfc5234).

## Restrictions

JAXN is mostly a superset of JSON in that every JSON text is a JAXN text that represents the same value, however the following points restrict which JSON texts are also JAXN:

* A document is considered valid when it validates against the JAXN grammar *and* when all additional restrictions are met.
* Duplicate names are not allowed in objects; the behaviour in the presence of duplicate names is implementation defined.
* The ASCII control character `%x7F` MUST NOT appear in a JAXN text (it may be part of a string or binary value when quoted appropriately).

## Comments

#### Examples

* `# single-line comment`
* `// single-line comment`
* `/* block comment */`

#### Grammar

```abnf
comment = c-line / c-block

c-line = c-begin-line *( c-char )

c-begin-line = %x23 / %x2F.2F ; # or //

c-char = %x09 / %x20-7E / %x80-10FFFF
                              ; Any HTAB or printable character

c-block = c-begin-block
          *( c-no-star / ( 1*c-star c-no-star-or-slash ) )
          c-end-block

c-begin-block = c-slash c-star
c-end-block = 1*c-star c-slash

c-slash = %x2F                ; /
c-star = %x2A                 ; *

c-no-star = %x09 / %x0A / %x0D /
            %x20-29 / %x2B-7E / %x80-10FFFF

c-no-star-or-slash = %x09 / %x0A / %x0D /
                     %x20-29 / %x2B-2E / %x30-7E / %x80-10FFFF

ws = *( %x20 /                ; Space
        %x09 /                ; Horizontal tab
        %x0A /                ; Line feed or New line
        %x0D /                ; Carriage return
        comment )             ; Comment
```

#### Semantics

Comments change the representation of data but have no effect on which data is represented.

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
JAXN allows `+NaN` and `-NaN` as alternatives for `NaN`, as well as `+Infinity` as an alternative for `Infinity`.

All other extensions are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.

The permissible magnitude and precision of numbers is implementation defined.
It must allow at least IEEE 754 double-precision floating point numbers.

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

m-d-char = *2d-quote ( %x09 / %x0A / %x0D / %x20-21 / %x23-7E / %x80-10FFFF )
m-s-char = *2s-quote ( %x09 / %x0A / %x0D / %x20-26 / %x28-7E / %x80-10FFFF )

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

unescaped = %x20-21 / %x23-26 / %x28-5B / %x5D-7E / %x80-10FFFF
```

#### Notes

* Each string (in a concatenation: individually) **MUST** be a sequence of Unicode characters.
* `\uXXXX` with UTF-16 surrogates **MUST** be handled before concatenation.
* Unpaired UTF-16 surrogates **MUST NOT** appear in the string representation.
* `\u{X...}` **MUST NOT** encode surrogates (i.e. the represented string is not allowed to contain UTF-16 surrogates).
* Multiline strings:
  * Are surrounded by three quotation marks (single or double quote) on each side and allow newline.
  * Can contain any sequence of non-control characters except for three matching closing quotation marks.
  * Do not interpret escape sequences, a backslash `\` is just a literal backslash.
  * A newline immediately following the opening delimiter is trimmed.
  * All other characters remain intact.
* Concatenations can mix single- and double-quoted strings as well as single- or multiline-strings.
* Concatenation is a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
  It happens before the final string is passed on from the parser.

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

* Binary data represents arbitrary byte sequences (not Unicode strings).
* Binary strings allow only "printable" ASCII characters, no control characters.
* No `\uXXXX` or `\u{...}` escape sequences allowed, instead:
* Escape sequence `\xXX` for arbitrary byte values.
* Concatenations can mix single- and double-quoted binary strings as well as hexdumped data.
* Concatenation is a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
  It happens before the final binary value is passed on from the parser.

## Unquoted Names in Objects

#### Synopsis

Allow identifiers as unquoted names in objects.

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

Names in objects are strings; the tokens `true`, `null`, and `false` are unambiguous shortcuts for `"true"`, `"null"`, and `"false"` when used within an object where a name is expected.
Strings in their role as names in objects can use the extended syntax for strings including the single-quoted variant, additional escape sequences, and string concatenation, however unquoted names in objects can **not** be concatenated.

Unquoted names in objects are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated, i.e. they are passed on as normal strings from the parser.

## Trailing Comma

#### Synopsis

Allow trailing commas in arrays and objects.

#### Examples

* `[ 1, 2, 3, ]`
* `{ foo: "Hello", bar: 42, }`

#### Grammar

```abnf
value-sep-opt = [ value-separator ]

array = begin-array
        [ value *( value-separator value ) value-sep-opt ]
        end-array

object = begin-object
         [ member *( value-separator member ) value-sep-opt ]
         end-object
```

#### Semantics

The additional commas have no semantics.

#### Notes

The above grammar does not allow for adjacent commas (`[1,,2]`), a leading comma (`[,1]`), or placing a comma in an empty array or object (`[,]`).

Trailing commas are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.

Copyright (c) 2017-2018 Daniel Frey and Dr. Colin Hirsch
