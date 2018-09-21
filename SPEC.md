# Draft IPLD Unixfs Spec

## Basic Structure

A Unixfs is either a file or a directory.

The top level IPLD object has at least two fields: `type` and `data`
and maybe a few other such as a version string or a set of flags.

The `type` field is either `file` or `dir`.

## IPLD `file`

A file object has the following fields.

  - `type`: String with value of `'file'`.
  - `data`: TODO: Define structure for file content data.
  - `size`: Integer. Cumulative size of `data`.

The `type` field must be set to `file`.

## IPLD `dir`

A directory object represents a directory.

The key of the map is a filename and is a CBOR text string encoded in UTF-8.
The value of the map is another CBOR map with the following standard fields:

  - `type`: String with value of `'dir'`.
  - `data`: TODO: Define structure for directory content data.
  - `size`: Integer. Cumulative size of all directories and files in `data`.

The `type` field set to `dir`.
