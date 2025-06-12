# create_pack.md

# Creating Pack Files Containing Multiple Objects

---

## Overview

The `create_pack.md` document details the process of creating pack files in Git, which are efficient container files that store multiple Git objects (such as commits, trees, and blobs) in a compressed format. This file fits within the "Object Graph and Packfiles" section of the documentation tree, which focuses on managing commit graphs, object sets, and packfile creation. Creating pack files is a crucial step in optimizing network transfer and local storage, especially when pushing or fetching multiple objects between repositories.

This document explains the key function involved in packfile creation, how it operates, and demonstrates usage examples. The packfile format and the encoding of individual objects are also outlined, providing a foundation for understanding how Git bundles objects for efficient transport and storage.

---

## Function Documentation

### `create_pack(objects)`

#### Purpose

Creates a Git pack file containing all objects specified by their SHA-1 hashes. The function builds the packfile header, encodes each object in the pack format, concatenates them, and appends a SHA-1 checksum of the entire packfile. The returned value is the raw byte content of the complete pack file, ready to be written to disk or sent over the network.

This function is essential for bundling multiple Git objects into a single compressed file, which is used in operations like pushing commits to a remote repository.

#### Parameters

- `objects` (`set` of `str`): A set of SHA-1 hash strings representing the Git objects to include in the packfile.

#### Preconditions

- Each SHA-1 hash in `objects` must correspond to a valid Git object accessible in the local object store.
- The objects should be sorted or uniquely identified as the packfile header requires a count of objects.

#### How It Works (Step-by-Step)

1. **Packfile Header Construction:**
   - The function starts by packing a fixed header with:
     - The magic string `b'PACK'`
     - The packfile version number (2)
     - The number of objects to include (length of `objects`)

2. **Encoding Objects:**
   - For each object SHA-1 in `objects` (sorted for consistency), the function calls `encode_pack_object(obj)` which:
     - Reads the object type and data.
     - Creates a variable-length header encoding the object type and size.
     - Compresses the object data using zlib.
     - Returns the concatenation of the header and compressed data.

3. **Concatenating Packfile Contents:**
   - The encoded objects are concatenated in order immediately following the header.

4. **Appending SHA-1 Checksum:**
   - A SHA-1 checksum of the entire packfile (header + body) is computed.
   - The checksum is appended to the packfile contents.

5. **Return:**
   - The complete byte string of the packfile is returned.

#### Example Usage

```python
# Assume you have a set of object SHA-1 hashes to pack:
objects_to_pack = {
    'e69de29bb2d1d6434b8b29ae775ad8c2e48c5391',  # Example blob hash
    '4b825dc642cb6eb9a060e54bf8d69288fbee4904',  # Example tree hash
    'aefddf2e1d3c7e8b0b88e4f9c7a2b33a3c1b0be1',  # Example commit hash
}

pack_data = create_pack(objects_to_pack)

# Write the pack data to a file (e.g., .git/objects/pack/pack-xxxx.pack)
with open('.git/objects/pack/pack-example.pack', 'wb') as f:
    f.write(pack_data)
```

---

### `encode_pack_object(obj)`

#### Purpose

Encodes a single Git object into the packfile object format. This includes creating a variable-length header that encodes the object type and size, followed by the zlib-compressed object data.

This function is called by `create_pack()` for each object to prepare the individual object data for inclusion in the packfile.

#### Parameters

- `obj` (`str`): The SHA-1 hash string of the Git object to encode.

#### How It Works (Step-by-Step)

1. **Read Object:**
   - Uses `read_object(obj)` to retrieve the object type (e.g., 'blob', 'tree', 'commit') and raw data bytes.

2. **Determine Object Type Number:**
   - Maps the object type to its corresponding packfile type number (e.g., 3 for blob, 2 for tree, 1 for commit).

3. **Prepare Header:**
   - Constructs the first header byte combining:
     - The object type (four bits)
     - The lower four bits of the object size
   - For the rest of the size, uses variable-length encoding with continuation bits.

4. **Compress Data:**
   - Compresses the object data using zlib compression.

5. **Return:**
   - Concatenates the variable-length header bytes and compressed data and returns as bytes.

#### Example Usage

```python
obj_sha1 = 'e69de29bb2d1d6434b8b29ae775ad8c2e48c5391'
packed_obj_bytes = encode_pack_object(obj_sha1)

# packed_obj_bytes now contains the encoded and compressed form of the object,
# ready to be concatenated in a packfile.
```

---

## ASCII Diagram: Structure of a Git Packfile

```
+-------------------+
| PACK Header       |  <-- 4 bytes: "PACK"
| Version Number    |  <-- 4 bytes (network byte order)
| Number of Objects |  <-- 4 bytes (network byte order)
+-------------------+
| Object 1          |  <-- Variable length header + compressed data
| Object 2          |
| ...               |
| Object N          |
+-------------------+
| SHA-1 Checksum    |  <-- 20 bytes SHA-1 of all previous bytes
+-------------------+
```

- Each object inside the packfile is encoded as:

```
+----------------------------+
| Variable-length header     |  <-- Encodes object type and size
+----------------------------+
| Zlib compressed object data|
+----------------------------+
```

---

## Summary

This document covers the creation of Git packfiles via the `create_pack()` function, which aggregates multiple Git objects into a single file with an efficient format suitable for storage and network transfer. It relies on the `encode_pack_object()` function to prepare each object correctly. Understanding these functions is fundamental for implementing Git push/pull protocols and managing the internal Git object storage efficiently.