# objects.md

## Overview

This document provides comprehensive technical reference material for handling Git objects within the pygit project. It covers essential operations including reading, writing, encoding, finding, and hashing Git objects. The functions documented here — such as `read_object`, `hash_object`, and `encode_pack_object` — form the backbone of Git object management, enabling efficient storage, retrieval, and transmission of repository data. This file is part of the broader "Objects and Packfiles" section in the documentation tree, which focuses on Git object internals and packfile creation, supporting advanced repository operations like commits, pushes, and diffs.

---

## Function Documentation

### `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object from the object store given a SHA-1 prefix. It returns a tuple containing the object’s type (e.g., "commit", "tree", "blob") and its decompressed data as bytes.

**Parameters:**  
- `sha1_prefix` (str): A string prefix of the object's SHA-1 hash (must be unique enough to identify a single object).

**Operation:**  
1. Uses `find_object` to locate the object file path corresponding to the SHA-1 prefix.  
2. Reads the compressed object file and decompresses it using zlib.  
3. Parses the header, which contains the object type and size separated by a null byte (`\x00`).  
4. Validates that the declared size matches the actual data length.  
5. Returns the object type and the raw data bytes.

**Usage Example:**
```python
obj_type, data = read_object("a1b2c3d4")
print(f"Object type: {obj_type}")
print(f"Object data (first 100 bytes): {data[:100]}")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given object data and optionally writes the object to the Git object store in compressed form.

**Parameters:**  
- `data` (bytes): Raw data of the Git object.  
- `obj_type` (str): Type of the object, e.g., "blob", "tree", or "commit".  
- `write` (bool, optional): If True (default), writes the compressed object to the object store.

**Operation:**  
1. Constructs a header string in the format: `<obj_type> <len(data)>`.  
2. Concatenates the header, a null byte, and the data to form the full object content.  
3. Computes the SHA-1 hash of the full content.  
4. If `write` is True, writes the compressed object to `.git/objects/` under a directory named by the first two hex digits of the hash and a file named by the remaining 38 hex digits.  
5. Returns the SHA-1 hash as a hex string.

**Usage Example:**
```python
with open("README.md", "rb") as f:
    file_data = f.read()
sha1 = hash_object(file_data, "blob")
print(f"Stored blob object with SHA-1: {sha1}")
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encodes a single Git object into the packfile format, which is used for efficient transmission and storage of multiple objects.

**Parameters:**  
- `obj` (str): SHA-1 hash string of the object to encode.

**Operation:**  
1. Reads the object type and data via `read_object`.  
2. Maps the object type to a numeric type code using an enumeration (`ObjectType`).  
3. Builds a variable-length header encoding the type and size of the object:  
   - The first byte encodes the lower 4 bits of the size and the type (shifted).  
   - If the size is larger than 4 bits, subsequent bytes encode the remaining size bits with continuation bits set accordingly.  
4. Compresses the raw object data using zlib compression.  
5. Returns the concatenation of the header bytes and compressed data.

**Usage Example:**
```python
pack_data = encode_pack_object("e69de29bb2d1d6434b8b29ae775ad8c2e48c5391")
print(f"Encoded pack object size: {len(pack_data)} bytes")
```

**ASCII Diagram: Pack Object Encoding Header Format**

```
+-------------------------------+
| Variable-length header bytes  |
|-------------------------------|
| Bits: 3 bits for type          |
|       4+ bits for size         |
|       Continuation bit for ext |
+-------------------------------+
| Followed by zlib compressed    |
| object data                   |
+-------------------------------+
```

---

### `find_object(sha1_prefix)`

**Purpose:**  
Locates the filesystem path of a Git object given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): The prefix of the SHA-1 hash (minimum 2 characters).

**Operation:**  
1. Ensures prefix length is at least 2 characters.  
2. Lists the files in the `.git/objects/` directory matching the first two characters of the prefix.  
3. Filters files starting with the remaining characters of the prefix.  
4. Raises `ValueError` if no matching objects or multiple matches are found.  
5. Returns the full path to the unique matching object file.

**Usage Example:**
```python
path = find_object("a1b2c3")
print(f"Object file path: {path}")
```

---

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object and returns a list of its entries, where each entry consists of the mode, path, and SHA-1 hash of the object referenced.

**Parameters:**  
- `sha1` (str, optional): SHA-1 hash of the tree object to read.  
- `data` (bytes, optional): Raw data of the tree object. Must provide one of `sha1` or `data`.

**Operation:**  
1. If `sha1` is given, reads the object data and verifies it is a tree.  
2. Iteratively parses entries from the raw data:  
   - Reads a mode and path string terminated by a null byte.  
   - Reads the 20-byte SHA-1 digest following the path.  
3. Returns a list of tuples `(mode, path, sha1)`.

**Usage Example:**
```python
entries = read_tree(sha1="4b825dc642cb6eb9a060e54bf8d69288fbee4904")
for mode, path, sha1 in entries:
    print(f"{mode:o} {path} {sha1}")
```

---

### `hash_object` and `read_object` Interaction Diagram

```
+-------------------+              +---------------------+
| Raw Data + Type   |              | SHA-1 Prefix String  |
| (blob, commit...) |              |                     |
+---------+---------+              +-----------+---------+
          |                                  |
          | hash_object(data, type)          | read_object(sha1_prefix)
          |                                  |
          v                                  v
+-------------------+              +---------------------+
| Compute SHA-1     |              | Find Object Path    |
| Compress Data     |              | Decompress Data     |
| Write to .git/obj |              | Parse Header        |
+---------+---------+              +-----------+---------+
          |                                  |
          +----------------------------------+
                        (SHA-1 Hash String)
```

---

### `hash_object` and `write_file`

**Note:** The `hash_object` function internally calls `write_file` to persist the compressed object data. The `write_file` function safely writes bytes to the specified path.

---

### `read_object` Usage in Higher Level Functions

Functions such as `cat_file`, `diff`, and `commit` rely on `read_object` to access the contents of Git objects for display, comparison, or commit creation.

---

## Summary

The `objects.md` file documents critical functions for:

- Reading Git objects by SHA-1 prefix (`read_object`)  
- Hashing and storing objects (`hash_object`)  
- Encoding objects for packfile transmission (`encode_pack_object`)  
- Locating objects on disk (`find_object`)  
- Parsing tree objects (`read_tree`)

These foundational operations enable higher-level Git commands and network protocols to function correctly, ensuring data integrity and efficient object management. The provided examples and diagrams clarify the data flow and usage patterns, aiding developers in understanding and extending pygit's object handling capabilities.