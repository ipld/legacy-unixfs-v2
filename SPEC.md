# Draft IPLD Unixfs Spec

## Basic Structure

A Unixfs is either a file or a directory.

The top level IPLD object has at least two fields: `type` and `data`
and maybe a few other such as a version string or a set of flags.

The `type` field is either `file` or `dir`.

## IPLD `file`

A file object has the following fields.

  - `type`: String with value of `'file'`.
  - `data`: Array of `file-data`.
  - `size`: Integer. Cumulative size of `data`.

The `type` field must be set to `file`.

### `file-data`

File data is an Array. Each element is an Array with only 2 elements (Tuple).

  - 0: Array. Tuple containing two integers, the `start` and `length` offsets of the content.
    - 0: Integer: start offset.
    - 1: Integer: length of part(s).
  - 1: Link. Either a `raw` link or a link to another node formatted as `file-data`.

```javascript
[
  [ [0, 1024], Link ],
  [ [1025, 1002], Link ]
]
```

Implementations are encouraged to use nested `file-data` array nodes through links for files
with many chunks in order to limit the size of the serialized node. No strict boundary is
set but it is typical to try and limit node size to less than one megabyte.

## IPLD `dir`

A directory object represents a directory.

The key of the map is a filename and is a CBOR text string encoded in UTF-8.
The value of the map is another CBOR map with the following standard fields:

  - `type`: String with value of `'dir'`.
  - `data`: [WIP](https://github.com/ipfs/unixfs-v2/pull/19)
  - `size`: Integer. Cumulative size of all directories and files in `data`.

The `type` field set to `dir`.
