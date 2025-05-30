# packfiles.md

# Creating and Encoding Pack Files for Efficient Object Storage and Transfer

---

## Overview

This document covers the creation and encoding of Git pack files, which are essential for efficient storage, transfer, and management of Git objects within the repository. Pack files consolidate multiple loose Git objects into a single compressed file, reducing disk space usage and network bandwidth during operations such as pushing to or fetching from remote repositories.

Within the broader documentation hierarchy, `packfiles.md` resides under the **Object Storage and Packfiles** section, complementing other object management files like `objects.md` and `object_reading.md`. This file specifically details the mechanisms for encoding objects into pack files and assembling complete pack files, supporting workflows such as `pygit.diff` and `pygit.push` where packed data transmission is critical.

---

## Functions

### `encode_pack_object(obj)`

#### Purpose

Encodes a single Git object into the pack file format, producing a bytes sequence that begins with a variable-length header and is followed by the zlib-compressed object data. This encoded form is a building block for constructing full pack files.

#### Parameters

- `obj` (str): The SHA-1 hash (hex string) of the Git object to encode.

#### Operation

1. Reads the Git object identified by `obj` (SHA-1 hash) from the object store:
   - Retrieves the object type (e.g., `blob`, `tree`, `commit`) and raw data bytes.
2. Maps the object type to its numeric representation (per Git packfile specification).
3. Calculates the size of the object data.
4. Constructs the first byte of the header:
   - Bits 4-6 encode the object type.
   - Bits 0-3 hold the lowest 4 bits of the size.
5. Encodes the remaining size using a variable-length format where each byte’s highest bit signals continuation:
   - For each 7 bits of the size remaining, a byte is emitted with the high bit set.
   - The last byte has the high bit cleared.
6. Concatenates the header bytes.
7. Compresses the object data using zlib.
8. Returns the concatenated header and compressed data bytes.

#### Example Usage

```python
# Encode a blob object with SHA-1 hash 'f572d396fae9206628714fb2ce00f72e94f2258f'
encoded_bytes = encode_pack_object('f572d396fae9206628714fb2ce00f72e94f2258f')

# The result can be written to a pack file as part of the pack body.
with open('example.pack', 'ab') as f:
    f.write(encoded_bytes)
```

---

### `create_pack(objects)`

#### Purpose

Creates a complete Git pack file containing a set of Git objects. The pack file includes a header, the concatenated encoded objects, and a trailing SHA-1 checksum for integrity verification.

#### Parameters

- `objects` (set of str): A set of SHA-1 hashes (hex strings) representing Git objects to include in the pack.

#### Operation

1. Packs the header consisting of:
   - The ASCII signature `PACK`.
   - The pack file version number (2 as per Git spec).
   - The number of objects included.
2. Sorts the `objects` set to ensure consistent ordering.
3. Encodes each object using `encode_pack_object`.
4. Concatenates all encoded objects to form the pack body.
5. Concatenates the header and body to form the pack contents.
6. Computes the SHA-1 checksum of the pack contents.
7. Appends the checksum to the pack contents.
8. Returns the complete pack file data as bytes.

#### Example Usage

```python
# Suppose we have a set of object hashes to pack
object_hashes = {
    'f572d396fae9206628714fb2ce00f72e94