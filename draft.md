# Draft Ipld Unixfs Spec

## Basic Structure

   - Some sort of header that indicates that this a directory and included a version number.  The header could also have fields to give additional information on the meaning of the extended attributes.

  - CBOR Map
    - Key: CBOR Byte or Text String: File Name
    - Value: CBOR Array of:
      - Type: CBOR Unsigned Int
      - Link or Data: CBOR Type varies
      - Optional file size: CBOR Unsigned Int
      - Optional Standard Attributes: CBOR Map
      - Optional Extended Attributes: CBOR Map

The file size is only defined for regular files and is the size of the file contents.

All maps should be ordered based on the binary values of the key,
duplicates are not allowed.

### Notes

 * An array makes sense to be as this is more compact and the value of
   the fields are unambiguous, it also allows for a separation of
   standard and extended attributes

 * The key type can either be a byte or text string as POSIX makes no
   requirements that file names be utf-8 and it is important that any
   file name can be faithfully represented, if the string is utf-8
   then the type will be Text.

## Types

The type field should be limited to a set of well defined values so it
makes sense that this is an integer rather than a text string.  The
value is the ascii value of a letter. When converting to JSON the
integer can be represented as a single character string.

Possible values are as follows:

  * 0, '', `file`: regular file
  * `e`, `exe`: executable file
  * `d`, `dir`: directory entry
  * `s`, `special`: special file type (fifo, device, etc). The second field is a CBOR Map with at least one field to describe the type.
  * `l`, `symlink`: symbolic link.  The second field is the contents of the link
  * `o`, `other`: link to other ipld object, links followed for GC and related operations 
  * `u`, `unknown`: link to unknown objects, links not followed

### Notes

  * Rather than have a special attribute for an executable bit it is more compact if we just make this a different type
  * It is very useful to be able to determine if a link is a directory or an ordinary file so I made it as separate type, also there can be multiple ways to define a file size for a directory so it is best to just leave it out as it is of limited usefulness

## Standard Attributes:

The standard set of attributes should be limited to a small set of meaningful values.
Stripping this filed SHOULD not change the meaning of the directory entry.
Clients SHOULD be able to understand these attributes when reading a directory entry.

Possible entries:

  * `mtime`
  * `ro`: Boolean, set if the file or directory should be readonly when copied to the filesystem

## Extended Attributes

The extended attributes set is not well defined and can be used for vendor extensions and posix attributes that don't make sense on non-unix systems.
Stripping this field MUST not change the meaning of the directory entry.
These attributes SHOULD be passed along but do not have to be understood.
The directory header MAY include information on the meaning of the attributes;
for example it could indicate that this is a copy of a unix filesystem and to expect a standard set of corresponding attributes.

Possible entries:

  * `user`: unix user name
  * `uid`: unix numeric uid
  * `group`: unix group name
  * `gid`: unix numeric gid
  * `perm`: full unix permissions
  * extended posix attributes
  * windows specific attributes
