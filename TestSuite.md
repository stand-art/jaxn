# JAXN Test Suite

> **Please note that this test suite does not yet exist**, this section is to discuss its development...

The JAXN test suite is intended to cover all aspects and details of the JAXN encoding.
A library that passes all tests can be considered JAXN compliant.
The test suite contains different categories of testcases, each consisting of one or more data files.

For the purpose of this document, two JSON files, or two JAXN files, are considered equivalent if they describe the same data.
This can be checked by first parsing and then printing the two files with *TBD tools in taocpp/json* and checking whether the outputs are equal.

## Invalid JAXN

Each testcase consists of one input file that does not conform to the JAXN standard.
Reading such a file must generate an error.

## JAXN encoded JSON

Each testcase consists of one valid JAXN input file, and one valid JSON reference file.
The JAXN input files in this category remain within the JSON data model.
The JAXN input file must be parsed and then printed to a JSON output file.
The test passes if the JSON output file and the JSON reference file are equivalent.

## JAXN beyond JSON

Each testcase consists of one valid JAXN input file, and one valid JAXN reference file.
The JAXN input file must be parsed and then printed to a JAXN output file.
The test passes if the JAXN output file and the JAXN reference file are equivalent.

Copyright (c) 2017-2018 Daniel Frey and Dr. Colin Hirsch
