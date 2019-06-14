# [Draft] IPLD UnixFS Spec

## Basic Structure

A Unixfs is either a file or a directory.

This specification uses the term "path(s)" to refer to specific key/value 
pairs. Every path could be either an in-line value or a Link. This means 
that a single UnixFS tree could be entirely inlined into a single Block or 
could be many Blocks connected by CID Links.

A top level IPLD object has at least two paths: `type` and `data`
and maybe a few other such as a version string or a set of flags.

UnixFSv2 uses a still experimental system for
[composite types](https://github.com/ipld/specs/pull/126). When `Bytes` 
and `Map` are used in this specification they refer not only to IPLD 
Data Model "kinds" but to any IPLD data structure that supports the
right operations.

## IPLD `file`

A file object has the following paths.

  - `type`: String with value of `'IPFS/Experimental/File/0'`. *Note: this string will change when the specification is ratified*.
  - `data`: `Bytes`.
  - `size`: Integer. Cumulative size of `data`.

## IPLD `dir`

A directory object represents a directory and has the following paths.

  - `type`: String with value of `'IPFS/Experimental/Dir/0'`. *Note: this string will change when the specification is ratified*. 
  - `data`: `Map`.
  - `size`: Integer. Cumulative size of all directories and files in `data`.

