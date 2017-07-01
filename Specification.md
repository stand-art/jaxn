# Specification

The following sections specify the syntax and semantics of the extensions that JAXN brings to JSON.

* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Unquoted Object Keys](#unquoted-object-keys)
* [Trailing Comma](#trailing-comma)
* [Binary Data](#binary-data)
* [ABNF for JAXN](#abnf-for-jaxn)

## Comments

#### Synopsis

Allows single-line and block comments.

### Single-line comments

#### Examples

* `# single-line comment`
* `// single-line comment`

#### Grammar

```abnf
c-line = c-begin-line *( c-char ) c-end-line

c-begin-line = %x23 / %x2F.2F ; # or //
c-char = HTAB / %x20-10FFFF   ; Any HTAB or printable character
c-end-line = eol / eof
```

#### Semantics

Single-line comments **MUST** be ignored. They **MUST NOT** be forwarded by a JAXN parser to the caller. They **MUST NOT** modify the sematics of the parsed JAXN document.

#### Notes

A single-line comment may not contain additional control characters. It ends at either the end of the line, or at the end of  input, whichever is encountered first.

### Block comments

#### Example

`/* block comment */`

#### Grammar

```abnf
c-block = c-begin-block *( c-no-asterisk / ( c-asterisk c-no-slash ) ) c-end-block

c-begin-block = %x2F.2A       ; /*
c-end-block = %x2A.2F         ; */

c-asterisk = %x2A             ; * asterisk

c-no-asterisk = eol / HTAB / %x20-29 / %x2B-10FFFF
                              ; Any eol, HTAB or printable character except *

c-no-slash = eol / HTAB / %x20-2E / %x30-10FFFF
                              ; Any eol, HTAB or printable character except /
```

#### Semantics

Block comments **MUST** be ignored. They **MUST NOT** be forwarded by a JAXN parser to the caller. They **MUST NOT** modify the sematics of the parsed JAXN document.

#### Notes

Block comments do not nested. In other words, occurrences of `/*` within a block comment are not interpreted as anyhting else other than part of the comment.

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

JAXN adds non-finite values to the data model that can not be represented in JSON. The spelling of the identifiers is case-sensitive. All other extensions are only extending the syntax. JAXN allows `+NaN` and `-NaN` as alternatives for `NaN`, as well as `+Infinity` as an alternative to `Infinity`.

## Strings

#### Synopsis

Allow single-quoted strings, additional escape sequences, and string concatenation.

#### Examples

* `"Add \0 or \v, even \' is allowed in a string."`
* `'That\'s right, you need to escape single-quotes in a single-quoted string.'`
* `'Oh, and \" is allowed even in a single-quote string.'`
* `"\u{1D11E} was my first love " + "and it will be my last."`

#### Grammar

```abnf
string = string-part *( value-concat string-part )

string-part = d-string / s-string

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
* Concatenations can mix single- and double-quoted strings.

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

Object keys are strings, wherefore the tokens `true`, `null`, and `false` are unambiguous shortcuts for `"true"`, `"null"`, and `"false"` when used in an object key position. Strings in object key positions can use the extended syntax for strings including the single-quoted variant, additional escape sequences and string concatenation, however unquoted object keys can **not** be concatenated.

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

(only showing the relevant differences against RFC 7159)

#### Semantics

The additional commas have no semantics.

#### Notes

The above grammar does not allow for adjacent commas (`[1,,2]`), a leading comma (`[,1]`), or placing a comma in an empty array or object (`[,]`).

## Binary Data

#### TODO: Add similar sections like for the other extensions.

* Binary data represents arbitrary byte sequences, not Unicode strings.
* Two syntactical variants that can be concatenated with each other.
* Hexdumped binary, e.g. `$48656c6c6f2c20776f726c6421`.
  * Allows optional dots, e.g. `$48.65.6c.6c.6f.2c.20.77.6f.72.6c.64.21`.
* Binary strings, e.g. `$"Hello, \x77orld!"`.
  * Only printable ASCII characters allowed, no control characters.
  * No `\uXXXX` or `\u{...}` escape sequences allowed, instead:
  * Add `\xXX` for arbitrary byte values.

## ABNF for JAXN

The grammar for well-formed JAXN is based on and extends the ABNF grammar given in the JSON RFC 7159.

It is also available as a separate [ABNF file](jaxn.abnf).

```abnf
eol = <end-of-line>           ; End-of-line
eof = <end-of-file>           ; End-of-file

comment = c-line / c-block

c-line = c-begin-line *( c-char ) c-end-line

c-begin-line = %x23 / %x2F.2F ; # or //
c-end-line = eol / eof

c-block = c-begin-block *( c-no-asterisk / ( c-asterisk c-no-slash ) ) c-end-block

c-begin-block = %x2F.2A       ; /*
c-end-block = %x2A.2F         ; */

c-char = HTAB / %x20-10FFFF   ; Any HTAB or printable character

c-asterisk = %x2A             ; * asterisk

c-no-asterisk = eol / HTAB / %x20-29 / %x2B-10FFFF
                              ; Any eol, HTAB or printable character except *

c-no-slash = eol / HTAB / %x20-2E / %x30-10FFFF
                              ; Any eol, HTAB or printable character except /

ws = *(
          SP /                ; Space
          HTAB /              ; Horizontal tab
          eol /               ; Line ending
	  comment             ; Comment
      )

begin-array     = ws %x5B ws  ; [ left square bracket
begin-object    = ws %x7B ws  ; { left curly bracket
end-array       = ws %x5D ws  ; ] right square bracket
end-object      = ws %x7D ws  ; } right curly bracket
name-separator  = ws %x3A ws  ; : colon
value-separator = ws %x2C ws  ; , comma
value-concat    = ws %x2B ws  ; + plus

value-sep-opt = [ value-separator ]

null  = %x6E.75.6C.6C         ; null
true  = %x74.72.75.65         ; true
false = %x66.61.6C.73.65      ; false

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

string = string-part *( value-concat string-part )

string-part = d-string / s-string

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

array = begin-array [ value *( value-separator value ) value-sep-opt ] end-array

object = begin-object [ member *( value-separator member ) value-sep-opt ] end-object

member = key name-separator value

key = string / identifier

identifier = i-begin *i-continue

i-begin = ALPHA / %x24 / %x5F
i-continue = i-begin / DIGIT

value = false / null / true / object / array / number / string / binary

JAXN-text = ws value ws
```

Copyright (c) 2017 Daniel Frey and Dr. Colin Hirsch
