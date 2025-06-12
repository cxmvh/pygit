# object_reading.md

## Overview

This document covers the mechanisms for reading and locating Git objects within the repository, focusing on the core functions `read_object`, `find_object`, and `read_tree`. These functions form the foundation for interacting with Git's object database, enabling retrieval and decoding of various object types such as commits, trees, and blobs. This file is part of the "Git Objects and Packfiles" section of the documentation tree, complementing related files like `cat_file.md` and `object_encoding.md`. Mastery of these functions is essential for understanding how Git stores and accesses its data internally.

---

## Function Documentation

### 1. `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object identified by a SHA-1 hash prefix from the object store. It returns a tuple containing the object's type (e.g., `commit`, `tree`, `blob`) and the raw data bytes of the object.

**Parameters:**  
- `sha1_prefix` (str): A prefix of the SHA-1 hash identifying the object. Must be at least two characters to locate the object folder.

**Operation:**  
- Calls `find_object` to locate the object file in the `.git/objects` directory corresponding to the given SHA-1 prefix.
- Reads and decompresses the object file.
- Parses the object header to extract the object type and size.
- Validates that the size matches the data length.
- Returns the object type and the raw data.

**Usage Example:**
```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(f"Data bytes length: {len(data)}")
```

---

### 2. `find_object(sha1_prefix)`

**Purpose:**  
Finds the filesystem path of a Git object given a SHA-1 prefix. Raises an error if no matching object or multiple matches are found.

**Parameters:**  
- `sha1_prefix` (str): SHA-1 hash prefix of the object to locate.

**Operation:**  
- Verifies the prefix length is at least two characters.
- Determines the directory based on the first two characters of the prefix.
- Lists files in the directory that match the remaining prefix.
- Ensures exactly one matching object is found.
- Returns the full path to the object file.

**Usage Example:**
```python
path = find_object('a1b2c3d4')
print(f"Object stored at: {path}")
```

---

### 3. `read_tree(sha1=None, data=None)`

**Purpose:**  
Reads and parses a Git tree object, returning a list of entries representing the contents of the tree.

**Parameters:**  
- `sha1` (str, optional): SHA-1 hash of the tree object to read. Mutually exclusive with `data`.
- `data` (bytes, optional): Raw data bytes of a tree object. Mutually exclusive with `sha1`.

**Operation:**  
- If `sha1` is provided, calls `read_object` to get the tree data and verifies the object type is `tree`.
- If `data` is provided directly, uses it for parsing.
- Iterates over the raw bytes to extract entries, each consisting of:
  - File mode (e.g., `100644` for blobs, `40000` for trees).
  - File path.
  - SHA-1 hash of the object.
- Returns a list of tuples `(mode, path, sha1)`.

**Usage Example:**
```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

---

## ASCII Diagram: Git Object Lookup Flow

```
+----------------+      sha1_prefix      +----------------------+
| Caller (e.g.    |--------------------->| find_object           |
| read_object)    |                       | - Locate object file  |
+----------------+                       +----------------------+
                                                  |
                                                  | object path
                                                  v
                                       +----------------------+
                                       | read_object           |
                                       | - Decompress object   |
                                       | - Parse header        |
                                       +----------------------+
                                                  |
                                                  | (obj_type, data)
                                                  v
                                       +----------------------+
                                       | Caller usage          |
                                       +----------------------+
```

---

### Additional Context: `cat_file` Function

The `cat_file` function uses `read_object` and `read_tree` to display object contents in various modes (raw, size, type, pretty). For example, in pretty mode for trees, it iterates over entries returned by `read_tree` and prints them with human-readable formatting.

---

## Summary

- `read_object` is the primary function to load and parse Git objects from the object store.
- `find_object` handles resolving the file path for a given SHA-1 prefix.
- `read_tree` specializes in parsing tree objects into their constituent entries.
- These functions underpin higher-level commands and utilities that manipulate Git objects.

This document should be referenced alongside `cat_file.md` and `object_encoding.md` for a comprehensive understanding of Git object handling inside the repository.