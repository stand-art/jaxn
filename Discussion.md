# Discussion

The following sections discuss the syntax and semantics of the extensions that JAXN brings to JSON, as well as rejected extensions that will not be added to JAXN.

* [Data Model](#data-model)
* [Unicode](#unicode)
* [White-Space](#white-space)
* [Newline](#newline)
* [Source Character Set](#source-character-set)
* [Comments](#comments)
* [Numbers](#numbers)
* [Strings](#strings)
* [Binary Data](#binary-data)
* [Unquoted Object Keys](#unquoted-object-keys)
* [Trailing Comma](#trailing-comma)
* [Conversion to JSON](#conversion-to-json)

## Data Model

Most "relaxed JSON" extensions focus on the syntax of the string representation.
They sometimes do extend the data model, but they don't say so clearly or are even unaware of it.
JAXN goes further, by clearly specifying which additional values and data types a library should support.
This allows users to know what to expect from a JAXN-compatible library, or, looking at it from the other side, search for a JAXN-compatible library when they know that they need these extensions to the data model.

JAXN extends the JSON data model in two places.

1. Allow `NaN`, `Infinity` and `-Infinity` for numeric values.
2. Add a binary data type.

## Unicode

JAXN does not require additional Unicode support beyond what is already required by JSON itself.
Some other libraries require additional Unicode support to implement certain extensions.
While this seems reasonable if you are working with a programming language or environment where Unicode support is available, it places an additional burden on others, e.g. in the embedded world.

A JAXN parser parses a sequence of bytes, the input data. The parser is...

* ...required to accept (correctly encoded) UTF-8 input data, this is the only interoperable representation.
* ...allowed to accept (correctly encoded) UTF-16 or UTF-32 input data.
* ...allowed to accept a byte order marker (BOM) at the begin of the input data.
* ...allowed to accept other encodings, provided that they are correctly identified (no guessing!) and unambiguously mapped to a sequence of Unicode code points.
* ...required to emit an error if it encounters an (encoding) error in the input data.

## White-Space

JAXN does not allow additional white-space characters.
Some other libraries allow additional white-space characters, but we do not see a real-world use-case for those.
We believe users often add them by mistake and this is not a good-enough reason for us to allow them.

## Newline

JAXN grammar allow well-formed documents to contain any sequence of 0x0A (Line feed or New line) and 0x0D (Carriage return) characters, mixed in any way, to be contained in the source data.
A JAXN parser is allowed (even expected) to further restrict the accepted end-of-line markers, for example to the system-native 0x0A (as it is common on Unix- and macOS-systems) or to require the sequence 0x0D, 0x0A (on Windows-systems).
This is necessary to report sensible position information in case of parse errors, as the line number in which an error occurs depends on the specific end-of-line markers allowed/expected for the input data.

## Source Character Set

The source character set (i.e., the Unicode code points that may be contained in the input data) consists of HTAB (0x09), the end-of-line characters (0x0A, 0x0D), and all code points from space upwards (0x20-0x10FFFF).
JSON allows 0x7F, although it is a control character.
JAXN stays compatible with JSON, so it has to allow it as well.

If a JAXN parsers encounters a code point outside of the source character set, it must report an error.

JAXN does not *require* any non-ASCII characters in the input data.
All Unicode code points in the string values in the data model can be written in an escaped form in the input data.
JAXN documents can therefore, like JSON documents, be restricted to ASCII without loosing expressiveness.

## Comments

JAXN allows comments, however, they are a presentation detail and must not have any effect on the serialization tree, representation graph or events generated.
In particular, comments are not associated with a particular node.
This improves interoperability and ensures that the main concern why comments are not part of JSON is taken care of.
The usual purpose of a comment is to communicate between the human maintainers of a file.
A typical example is comments in a configuration file.

Michael Bolin writes:

> Because JSON is more concise than XML, JSON is often a better format for data files that are maintained by hand. Examples include configuration files, as well as blobs of test data for web applications. For files such as these, it is convenient to be able to temporarily comment out bits of information (such as a configuration option, or an old test value in lieu of a new one). Further, if the file is to be maintained by humans, it is desirable to be able to include comments so that maintainers may communicate amongst one another without interfering with the data in the file.
>
> [...]
>
> This begs the question: why aren't comments officially supported in JSON? Interestingly, when Douglas Crockford originally introduced JSON, there was explicit support for C-style comments. He [later dropped support for them in the specification](http://tech.groups.yahoo.com/group/json/message/156), but also [declared that a JSON decoder that accepts comments should be considered a valid JSON decoder](http://tech.groups.yahoo.com/group/json/message/152).

(Source: http://bolinfest.com/essays/json.html)

Note that comments sometimes don't interact nicely with strings.
If you try to comment out a parts of a document that contains strings, and if those strings contain the character sequence `*/`, using a block comment will fail.
This problem of block comments existed long before JAXN.
As JSON already allows escaping the slash with a backslash in strings, you might consider converting `*/` into `*\/` within the string in question, you will then be able to comment the string out (and in again) without problems.

The restrictions on the source character set also apply within comments.

## Numbers

JAXN allows non-finite floating point values.
NaN and Infinity (as well as -Infinity) are well known, non-finite values from IEEE 754.
Real-world use-cases often require to deal with those values and providing a clear way to handle those non-finite values improves interoperability.
A JAXN-compatible library is required to accept NaN and Infinity as valid numeric values for their internal data model.

## Strings

JAXN keeps the JSON string data model intact.
String values in JSON are required to be valid Unicode strings in order to be interoperable.
The JSON RFC 8259 explains in paragraph 8.2 why this is the case.
JAXN does *not* change this.
Unlike some other libraries that allow escape sequences like `\xXX` for normal strings without specifying the semantics (properly), JAXN does not create ambiguity and confusion and does not require to store non-Unicode strings.

(I cleaned up this section a bit, as parts are moved to the above "Unicode" section.
We might still need to add some details and clarifications here, but let's not get overboard...)

The sequence of represented Unicode code points is obtained from the sequence of representation code points by replacing escape sequences with the escaped code points (or, in the case of UTF-16 surrogates, temporary code units), and by merging the code units of subsequent high and low UTF-16 surrogates into a single code point.

> (RFC 8259 specifies how to encode code points not in the BMP with a 12-character encoding consisting of two `\uXXXX` escape sequences using UTF-16 surrogate pairs, but does not mandate a specific behaviour when the merging of surrogates fails, noting only that it could be "unpredictable" including "fatal".)

There appears to be an implicit silent consensus among JSON libraries to ignore unpaired surrogates.
For now the JAXN standard allows unpaired surrogates.
Therefore, at level 3, not all represented strings are actually valid sequences of unicode characters.
(If the implementation uses UTF-8 as encoding, the final string encoding on level 4 can contain UTF-8 sequences for UTF-16 surrogates.)
A JAXN implementation MAY choose to forbid unpaired surrogates, a behaviour compatible with RFC 8259.

Merging of surrogate pairs, and the decision of whether a string contains unpaired surrogates, MUST be performed before concatentation of strings.

## Binary Data

In real-world uses, one often needs to handle binary data.
Representing this kind of data as strings requires, for example, hex- or base64-encoding.
As JAXN recognizes the importance of binary data, we extend the data model of a JAXN-compatible library by an explicit binary type.
For the representation in string form, we have chosen hex notation as base64 is human-unfriendly and adds additional implementation complexity.
Having a binary type and a more direct representation allows for a more consise and reasonable representation.

Implementations must treat binary data as a separate data type.
This increases interoperability with binary protocols like CBOR as well as providing a clear separation of readable strings from binary data.
The latter is helpful when you are dumping data to, say, a log-file.

## Unquoted Object Keys

Quoting Michael Bolin:

> Unfortunately, even though quoting is the minority case, JSON requires that all keys in maps must be double-quoted, regardless of whether they would need to be in ordinary JavaScript. Presumably this was done because it was the simplest way to guarantee that JSON would be a strict subset of ES3. (Fortunately, ES5 has evolved to allow JavaScript keywords to serve as unquoted property names in object literals.)
>
> Similar to the situation with trailing commas, if the design of JSON were not encumbered by the shortcomings of ES3, then I imagine that JSON keys would not have to be quoted. [...] In either case, demoting quoting from a requirement to an option would save most developers two bytes per key, which would be a win for both humans and machines. (I also think that it would make JSON more readable, though that may be a personal preference.)

(Source: http://bolinfest.com/essays/json.html)

## Trailing Comma

Again, Michael Bolin provides a good rationale for trailing commas:

> Most modern browsers allow for a trailing comma in array and object literals in JavaScript. Although support for the trailing comma was not mandated until ES5, browsers such as Chrome and Firefox have supported it for a long time
>
> [...]
>
> Using the trailing comma is particularly convenient for developers who may modify the map in the course of development. As shown in the following example, commenting out the last entry in a map can inadvertently transform it into an object literal with a trailing comma:

```json
// Commenting out the last line produces an object literal with a
// trailing comma.
{
  "margin": "2px",
  // "padding": "3px"
}
```

(Source: http://bolinfest.com/essays/json.html)

## Conversion to JSON

A JAXN data value may contain values that have no direct representation in JSON.
Those are the non-finite numeric values and binary data.
A library may chose to report an error when conversion to JSON string representation is requested.
If may also chose to replace those values with strings.
A JAXN-compatible library should use the following strings:

* `"NaN"` for a NaN. No other strings should be used, e.g. `"nan"`, `"+NaN"` or `"-NaN"`.
* `"Infinity"` and `"-Infinity"`. No other strings should be used, e.g. `"Inf"`, `"+Infinity"`, etc.
* Binary data should be represented as a string containing the hex encoded data, e.g. `"496E66696E697479"`.

Copyright (c) 2017-2018 Daniel Frey and Dr. Colin Hirsch
