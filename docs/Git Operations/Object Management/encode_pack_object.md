# encode_pack_object.md

# Encoding Git Objects into Pack File Format for Efficient Transfer

---

## Overview

This document explains the process and implementation details of encoding Git objects into the pack file format. Pack files are used by Git to efficiently transfer multiple objects between repositories, significantly reducing data size and improving performance during operations like pushing and fetching. The `encode_pack_object` function is central to creating these pack files by encoding individual Git objects with a variable-length header and compressed data. This file fits within the broader "Object Management" section of the documentation tree and supports the `pygit.push` operation by preparing objects for network transmission.

---

## Function Documentation

### `encode_pack_object(obj)`

#### Purpose

Encodes a single Git object, identified by its SHA-1 hash, into the Git pack file format. This involves creating a variable-length header that encodes the object type and size, followed by the object's compressed content. This format is required when creating pack files for efficient transmission to remote Git repositories.

#### Parameters

- `obj` (`str`): The SHA-1 hash (hex string) of the Git object to encode.

#### Returns

- `bytes`: The encoded byte sequence representing the packed object, composed of the variable-length header and compressed object data.

#### Preconditions

- The object must exist in the local Git object database.
- The object is readable via the `read_object` function.
- The `ObjectType` enumeration is available, mapping Git object types to integer codes.

#### How It Works

1. **Read Object Data**: Given the object SHA-1 hash, the function reads the object type (e.g., `commit`, `tree`, `blob`) and the raw data using `read_object(obj)`.

2. **Determine Object Type Code**: Each Git object type corresponds to a numeric code:
   
   | Object Type | Code |
   |-------------|------|
   | commit      | 1    |
   | tree        | 2    |
   | blob        | 3    |
   | tag         | 4    |
   
   (Assuming the enumeration `ObjectType` provides these.)

3. **Compute Size and Prepare Header**:
   - The size of the object data is extracted.
   - The first header byte encodes:
     - Bits 4-6: object type code.
     - Bits 0-3: lowest 4 bits of the object size.
     - Bit 7: continuation bit (1 if more size bytes follow).
   - The size is shifted right by 4 bits.
   - Subsequent size bytes are encoded 7 bits at a time with the continuation bit set until all size bits are consumed.
   
4. **Build Header Bytes**:
   - The header bytes are collected in a list, with the last byte having the continuation bit unset (0).
   
5. **Compress Data**:
   - The raw object data is compressed using zlib compression.
   
6. **Concatenate Header and Compressed Data**:
   - The function returns the concatenation of the header bytes and the compressed data.

#### Example

```python
# Example usage of encode_pack_object

# Suppose you have an object SHA-1 hash
obj_sha1 = "a1b2c3d4e5f678901234567890abcdef12345678"

# Encode the object into pack file format bytes
packed_bytes = encode_pack_object(obj_sha1)

# packed_bytes now contains the variable-length header and compressed object data
# which can be concatenated with other packed objects and written into a pack file.
```

#### ASCII Diagram: Pack Object Encoding Header Structure

```
+-------------------------------------------------+
| First Header Byte                                |
| +---------------------------------------------+ |
| | Bits 7   | Bits 6-4         | Bits 3-0      | |
| | (cont)  | (object type)    | (size low 4)  | |
| +---------------------------------------------+ |
+-------------------------------------------------+
| Subsequent Size Bytes (if needed)               |
| +---------------------------------------------+ |
| | Bit 7 (cont) | Bits 6-0 (size bits)         | |
| +---------------------------------------------+ |
| ...                                             |
+-------------------------------------------------+
| Compressed Object Data                           |
+-------------------------------------------------+
```

- **Bit 7 (cont)**: Continuation bit; 1 if more size bytes follow, 0 if last.
- **Bits 6-4**: Encoded object type.
- **Bits 3-0**: Low 4 bits of the object size.
- **Subsequent bytes**: 7 bits of size each, with continuation bits.

---

## Related Functions in Context

- **`read_object(sha1_prefix)`**: Reads the raw object data and type.
- **`create_pack(objects)`**: Bundles multiple encoded objects into a pack file, prepending a header and appending a checksum.
- **`push(git_url, username, password)`**: Uses `create_pack` which internally calls `encode_pack_object` to prepare objects for sending to a remote repository.

---

## Summary

The `encode_pack_object` function is a crucial component in preparing Git objects for packed transmission. By encoding the object type and size in a compact variable-length header and compressing the object data, it enables the creation of pack files that are both space-efficient and compatible with Git's network protocols. This encoding step ensures that pushing changes to remote repositories is optimized for speed and bandwidth.