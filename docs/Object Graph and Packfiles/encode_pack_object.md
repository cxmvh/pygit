# encode_pack_object.md

# Encoding a Single Object for Pack Files

---

## Overview

The `encode_pack_object.py` module is responsible for encoding individual Git objects into the packed object format used within Git packfiles. Packfiles are compressed collections of Git objects designed to optimize storage and network transfer. This file plays a crucial role in the **Object Graph and Packfiles** section of the repository documentation, complementing other modules that manage packing multiple objects, finding missing objects, and handling object graphs.

Encoding a single object involves creating a variable-length header that encodes the object type and size, followed by the compressed object data. This process is a fundamental step before assembling packfiles containing multiple objects. Understanding this encoding method is essential for developers working on Git internals, particularly those dealing with storage optimization and transfer efficiency.

---

## Function Documentation

### `encode_pack_object(obj)`

Encodes a single Git object for inclusion in a packfile.

#### Purpose

This function takes the SHA-1 hash of a Git object, reads the object's content and type, and then encodes it into the packfile object format. This format consists of:

- A variable-length header encoding the object type and size.
- The compressed (zlib) object data.

The returned bytes can then be concatenated with other encoded objects to form a packfile.

#### Parameters

- `obj` (str): The SHA-1 hash (hex string) of the Git object to encode.

#### Returns

- `bytes`: The encoded object data ready for inclusion in a packfile.

#### Preconditions

- The object with the specified SHA-1 must exist in the Git object store.
- The object type must be one of the standard Git object types (`commit`, `tree`, `blob`, `tag`).

#### Operation Details

1. **Read the object**: The function calls `read_object(obj)` to obtain the object's type (e.g., `blob`) and its raw data bytes.

2. **Determine the object type number**: Each Git object type is mapped to a numeric value (`commit=1`, `tree=2`, `blob=3`, `tag=4`) as per the `ObjectType` enumeration.

3. **Construct the header**:
    - The header encodes the object type and the size of the object in a variable-length format.
    - The first byte contains:
        - The object's type shifted left by 4 bits.
        - The lower 4 bits of the object's size.
    - If the size is larger than 4 bits, subsequent bytes encode the remaining size bits, each with the most significant bit set (continuation bit), except the last byte.
    
4. **Compress the data**:
    - The raw object data is compressed using zlib compression.

5. **Concatenate**:
    - The header bytes and the compressed data bytes are concatenated and returned.

#### Example Usage

```python
from pygit import encode_pack_object

obj_sha1 = 'a1b2c3d4e5f678901234567890abcdef12345678'  # Example SHA-1
encoded_bytes = encode_pack_object(obj_sha1)

# encoded_bytes now contains the packfile-ready representation of the object.
# This can be written into a packfile or sent over the network.
print(f"Encoded object size: {len(encoded_bytes)} bytes")
```

#### ASCII Diagram: Packfile Object Encoding Structure

```
+---------------------------------------------+
| Variable-Length Header                       |
| (object type, size encoded in bytes)        |
+---------------------------------------------+
| zlib Compressed Object Data                  |
| (raw object data compressed with zlib)      |
+---------------------------------------------+
```

The header encodes the object type and size in a compact format to save space. Following the header, the object data is compressed to further reduce packfile size.

---

## Related Functions and Concepts

- `read_object(sha1_prefix)`: Reads a Git object from the object database, returning its type and raw data.
- `ObjectType`: Enumeration mapping Git object types to numeric codes.
- `zlib.compress(data)`: Compresses data using the zlib compression algorithm.
- `create_pack(objects)`: Uses `encode_pack_object` to encode multiple objects and assemble a complete packfile.

---

# Appendix: Source Code Reference

```python
def encode_pack_object(obj):
    """Encode a single object for a pack file and return bytes (variable-
    length header followed by compressed data bytes).
    """
    obj_type, data = read_object(obj)
    type_num = ObjectType[obj_type].value
    size = len(data)
    byte = (type_num << 4) | (size & 0x0f)
    size >>= 4
    header = []
    while size:
        header.append(byte | 0x80)
        byte = size & 0x7f
        size >>= 7
    header.append(byte)
    return bytes(header) + zlib.compress(data)
```

---

This documentation provides a detailed explanation of the `encode_pack_object` function essential for understanding how individual Git objects are prepared for inclusion in efficient, compressed packfiles.