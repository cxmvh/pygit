# object_encoding.md

## Overview

This document provides detailed information about encoding Git objects, particularly focusing on how Git objects are prepared for inclusion in pack files and how they are hashed for storage and identification. It fits within the "Git Objects and Packfiles" section of the broader documentation tree, complementing related files that cover reading, writing, and handling Git objects, as well as packfile creation. Understanding object encoding is essential for grasping how Git efficiently stores and transmits repository data.

---

## Function Documentation

### encode_pack_object(obj)

#### Purpose

Encodes a single Git object for inclusion in a pack file. The function reads the object data, creates a variable-length header encoding the object type and size, then compresses the object's data. The resulting bytes combine the header and compressed data, ready for concatenation into a packfile.

#### Parameters

- `obj` (str): The SHA-1 hex string identifying the Git object to encode.

#### Operation Details

1. Read the object identified by `obj` using the `read_object` function, which returns the object type (e.g., commit, tree, blob) and raw data bytes.
2. Convert the object type string to a numeric code using the `ObjectType` enumeration.
3. Prepare the size of the data for encoding.
4. Construct the first byte of the header:
   - The high 4 bits encode the object type.
   - The low 4 bits encode the least significant 4 bits of the size.
5. Right-shift the size by 4 bits to process the remaining size bits.
6. If size remains, encode it in a variable-length quantity:
   - For each 7 bits of the remaining size, a header byte is added with the high bit set.
7. Append the final size byte without the high bit set.
8. Concatenate the header bytes with the zlib-compressed object data.
9. Return the concatenated bytes.

#### Usage Example

```python
# Encode a blob object with SHA-1 hash 'a1b2c3...'
pack_bytes = encode_pack_object('a1b2c3d4e5f67890123456789abcdef123456789')
# pack_bytes now contains the variable-length header plus compressed data bytes
```

#### ASCII Diagram: Variable-Length Header Encoding

```
+----------------+-----------------+-----------------+----------------+
| 1st Header Byte | 2nd Header Byte | ... (optional)  | Last Header Byte|
+----------------+-----------------+-----------------+----------------+
| 4 bits: type   | 7 bits: size bits| 7 bits: size bits| 7 bits: size bits|
| 4 bits: size LSB| (high bit=1)     | (high bit=1)    | (high bit=0)    |
+----------------+-----------------+-----------------+----------------+

- The first byte encodes object type and the lowest 4 bits of size.
- Subsequent bytes encode the remaining size bits in 7-bit chunks.
- High bit set (1) means more bytes follow; last byte high bit cleared (0).
```

---

### hash_object(data, obj_type, write=True)

#### Purpose

Computes the SHA-1 hash of a Git object given its raw data and type. Optionally writes the object to the Git object store in its compressed form.

#### Parameters

- `data` (bytes): Raw bytes of the object content.
- `obj_type` (str): Type of the object (e.g., `'commit'`, `'tree'`, `'blob'`).
- `write` (bool): If `True`, writes the object to the object store.

#### Operation Details

1. Create the object header as a byte string in the format: `"{obj_type} {len(data)}\0"`.
2. Concatenate the header and the data to form the full object representation.
3. Compute the SHA-1 hash of the full object.
4. If `write` is `True`, compress the full object data using zlib, and write it to the path `.git/objects/XX/YYYY...` where `XX` is the first two hex digits of the SHA-1 and `YYYY...` is the remaining 38 hex digits.
5. Return the SHA-1 hash as a hexadecimal string.

#### Usage Example

```python
blob_data = b"Hello, Git!"
sha1_hash = hash_object(blob_data, 'blob')
print(f"Object hash: {sha1_hash}")
# This will write the compressed object to .git/objects/...
```

---

### cat_file(mode, sha1_prefix)

#### Purpose

Outputs information or content about a Git object identified by its SHA-1 prefix, depending on the mode requested.

#### Parameters

- `mode` (str): Determines the output:
  - `'commit'`, `'tree'`, `'blob'`: Output raw object data if type matches.
  - `'size'`: Output size of object data.
  - `'type'`: Output object type.
  - `'pretty'`: Output a prettified representation depending on object type.
- `sha1_prefix` (str): SHA-1 prefix string identifying the object.

#### Operation Details

1. Use `read_object` to retrieve the object type and data.
2. For `'commit'`, `'tree'`, `'blob'` modes:
   - Check that the object type matches the mode.
   - Write raw data bytes to stdout buffer.
3. For `'size'` mode:
   - Print the length of the data.
4. For `'type'` mode:
   - Print the object type.
5. For `'pretty'` mode:
   - For `'commit'` or `'blob'`, output raw data.
   - For `'tree'`, parse the tree and print its entries with mode, type, SHA-1, and path.
6. Raise errors for invalid modes or unexpected object types.

#### Usage Example

```python
# Display the type of the object with prefix 'a1b2c3'
cat_file('type', 'a1b2c3')

# Pretty-print the tree object with prefix 'deadbeef'
cat_file('pretty', 'deadbeef')
```

---

### read_object(sha1_prefix)

#### Purpose

Reads an object from the object store using its SHA-1 prefix and returns its type and raw data.

#### Parameters

- `sha1_prefix` (str): SHA-1 prefix of the object to read.

#### Operation Details

1. Locate the object file in `.git/objects/`.
2. Read and decompress the file contents using zlib.
3. Parse the header which is of the form: `"<type> <size>\0"`.
4. Extract the object type and size.
5. Extract the data following the header.
6. Verify that the size matches the length of the data.
7. Return a tuple `(object_type, data_bytes)`.

#### Usage Example

```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(f"Data length: {len(data)} bytes")
```

---

### Supporting Function: find_object(sha1_prefix)

#### Purpose

Finds the filesystem path of a Git object file given its SHA-1 prefix.

#### Parameters

- `sha1_prefix` (str): SHA-1 prefix string.

#### Operation Details

1. Ensure prefix length is at least 2 characters.
2. Navigate to `.git/objects/XX/` where `XX` is the first two characters of the prefix.
3. Search for files in this directory that start with the remaining prefix characters.
4. Raise errors if no matches or multiple matches found.
5. Return the full path to the object file.

---

# Additional Notes

- The encoding of objects for packfiles is critical for Git's efficiency in storing and transferring repository data.
- The variable-length header format enables compact representation of object metadata.
- The use of zlib compression reduces storage size.
- These encoding mechanisms are foundational for packfile creation (`create_pack`) and pushing objects to remote repositories.
- Understanding the encoding supports advanced Git operations such as object traversal, differential packing, and network transport.

---

# Summary ASCII Diagram: Encoding Flow for Pack Object

```
Object SHA-1 (hex) --> read_object() --> (type, data)
          |
          v
Encode Header (type + size) ---+
                               |--> Concatenate header + zlib.compress(data)
                               |
                               +--> Return bytes for packfile
```

---

# References to Related Documentation Files

- `object_storage.md`: details on reading, writing, and hashing Git objects.
- `packfiles.md`: covers packfile creation and management.
- `object_reading.md`: expanded information on reading Git objects and trees.

---

This documentation aims to clarify the encoding process for Git objects, a fundamental step in packfile construction and object storage within the Git system.