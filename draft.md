# Draft IPLD Unixfs Spec

## Basic Structure

A Unixfs is either a file or a directory.
The top level IPLD object is a CBOR map with at least two fields: `type` and `data`
and maybe a few other such as a version string or a set of flags.
The `type` field is either `file` or `dir`.

## IPLD `file`

If an IPLD file is a leaf its CID type is `raw` (0x55) and has no structure.
Otherwise its CID type is `dag-cbor` (0x71).
The `type` field is set to `file` and the `data` field is an CBOR array.
Each element of the array is CBOR map with the following fields:

  - `data`: link
  - `size`: cumulative size of `data`
  - `fsize`: (file size) cumulative size of the payload of `data`
  
The `fsize` field is omitted if the link is `raw` as it is the same value as size.

## IPLD `dir`

An IPLD `dir` represents a directory.
Its CID type is `dag-cbor` (0x71).
The `type` field set to `dir` and the data field is an CBOR map.
The key of the map is a filename and is a CBOR text string encoded in UTF-8.
The value of the map is another CBOR map with the following standard fields:

  - `type`
  - `exe`: CBOR boolean: executable bit
  - `data`: normally a CBOR link, but can be other types depending on the value of the `type` field
  - `size`: cumulative size of `data`
  - `fsize`: (file size) cumulative size of the payload of `data`
  - `fname`: CBOR byte string: original filename if it differs from the key

And at least the following optional fields:

  - `ro`: CBOR boolean: read only
  - `mtime`: Modification time
  - `attr`: CBOR Map: Extended attributes

Additional fields may be defined.  All implementation specific or user
defined fields should be stored under the `attr` field. 

### Directory Types

The type field is limited to a set of well defined values:

  * _omitted_: regular file 
  * `dir`: directory entry
  * `special`: special file type (fifo, device, etc).
     The `data` field is a CBOR Map with at least one field to describe the type.
  * `symlink`: symbolic link.  The `data` field is the contents of the link.
  * `other`: link to other IPLD object, links followed for GC and related operations 
  * `unknown`: link to unknown objects, links not followed

### Extended Attributes

The extended attributes set is not well defined and can be used for vendor extensions and POSIX attributes that don't make sense on non-unix systems.
Stripping this field MUST not change the meaning of the directory entry.
These attributes SHOULD be passed along but do not have to be understood.

Possible entries:

  * `user`: unix user name
  * `uid`: unix numeric uid
  * `group`: unix group name
  * `gid`: unix numeric gid
  * `perm`: full unix permissions
  * extended posix attributes
  * windows specific attributes

### Notes

* Note all standard fields need to be defined for all files types.

  * The `type` field is omitted for regular files.
  * The `exe` field is only present when true and only makes sense for regular files 
  * The `size` and `fsize` are only required when the type is a regular file and possibly a `dir`.
    For other types they may be defined if they have a meaningful value. 
  * The `fsize` field is omitted for files that are leaves (i.e. `raw`) as it is the same value as `size`.

* IPLD filenames must valid UTF-8 strings which the following additional constraints:
  (1) cannot contain the null (0x00) or `/` characters
  (2) cannot be the strings: `.` or `..`
  Other restricts may be put in place.
  If the original filename does not meet these requirements then an implementation MAY transform the file from
  the original, so it is valid IPLD file, and store the original file in the `fname` field.
  When extracting a directory to the filesystem an implementation
  MAY make use of `fname` to restore the original name.
  Implementations SHOULD reject invalid files with invalid names by default
  and only translate files when a special flag is given.
  When extracting implications SHOULD use the IPLD name and not `fname` unless a special flag is given.

* To save space fields of a directory may be assigned integer values.
  Integers have the added benefit of conveying additional meaning based on there values;
  for example, to distinguish between standard and optional fields.

* The `type` field may also be assigned integer values.
