# [Draft] IPLD UnixFS Spec

## Basic Structure

A Unixfs is either a file or a directory.

This specification uses the term "path(s)" to refer to specific key/value 
pairs. Every path could be either an in-line value or a Link. This means 
that a single UnixFS tree could be entirely inlined into a single Block or 
could be many Blocks connected by CID Links.

A top level IPLD object has at least two paths: `type` and `data`
and maybe a few other such as a version string or a set of flags.

The `type` path's value is either `file` or `dir`.

## IPLD `file`

A file object has the following paths.

  - `type`: String with value of `'file'`.
  - `data`: Array of `file-data`.
  - `size`: Integer. Cumulative size of `data`.

The `type` path must be set to `file`.

### `file-data`

File data is an Array. Each element is an Array with only 2 elements (Tuple).

  - 0: Array. Tuple containing two integers, the `start` and `length` offsets of the content.
    - 0: Integer: start offset.
    - 1: Integer: length of part(s).
  - 1: Binary or `file-data`. Value can be either the binary data or additional nested `file-data` structures.

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

A directory object represents a directory and has the following paths.

  - `type`: String with value of `'dir'`.
  - `data`: 
    - Object (or `hamt`) with keys as file/directory names that are string encoded in UTF-8.
    - Values must be `file` or `dir` nodes.
    - 
  - `size`: Integer. Cumulative size of all directories and files in `data`.

The `type` path must be set to `dir`.

When the keyspace is too large for a single node `data` should be a link to `hamt` node. `hamt` has 
not yet been specified but will follow the implementation [here](https://github.com/ipfs/go-hamt-ipld).
