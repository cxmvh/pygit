# object_reading.md

# Reading Git Objects from the Object Store

---

## Overview

This document details the procedures and functions involved in reading Git objects from the Git object store. It covers locating Git objects by their SHA-1 hash prefixes, decompressing their stored content, and parsing the raw object data into meaningful components such as commits, trees, and blobs. This file is part of the broader "Git Object Handling" section, which provides comprehensive documentation on managing Git objects including hashing, reading, finding, encoding, and interpreting these objects. The functions described here play a critical role in enabling commands like `git cat-file` (`pygit.cat_file`) to display object contents correctly.

---

## Important Functions

### 1. `read_object(sha1_prefix)`

**Purpose:**  
Reads a Git object identified by a SHA-1 prefix from the object store, returning its type and raw data bytes.

**Parameters:**  
- `sha1_prefix` (str): A hex string prefix of the object's SHA-1 hash. Must be at least 2 characters.

**Operation:**  
1. Locate the object file path in the `.git/objects` directory matching the SHA-1 prefix using `find_object`.
2. Read the compressed object file content.
3. Decompress the content using zlib.
4. Parse the header of the decompressed data, which contains the object type and size, separated by a null byte.
5. Extract the object data bytes following the header.
6. Verify that the size in the header matches the length of the data bytes.
7. Return a tuple of `(object_type, data_bytes)`.

**Usage Example:**
```python
obj_type, data = read_object('a1b2c3d4')
print(f'Object type: {obj_type}')
print(f'Object data (bytes): {data[:50]}')  # Print first 50 bytes of object data
```

---

### 2. `find_object(sha1_prefix)`

**Purpose:**  
Finds the file path of a Git object in the object store given a SHA-1 prefix.

**Parameters:**  
- `sha1_prefix` (str): A prefix of the full SHA-1 hash (minimum 2 characters).

**Operation:**  
1. Verify that the prefix length is at least 2.
2. Use the first two characters as the directory name inside `.git/objects`.
3. List files in this directory and match those starting with the remaining SHA-1 characters.
4. Raise an error if no objects or multiple objects match.
5. Return the full path to the matching object file.

**Usage Example:**
```python
path = find_object('a1b2c3d4')
print(f'Object path: {path}')
```

---

### 3. `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object to extract its entries (files and subtrees).

**Parameters:**  
- `sha1` (str, optional): SHA-1 hash of the tree object.
- `data` (bytes, optional): Raw data bytes of the tree object. Must specify either `sha1` or `data`.

**Operation:**  
1. If `sha1` is provided, read the corresponding object and verify that it is a tree.
2. If `data` is provided, parse it directly.
3. Iterate over entries in the tree data:
   - Each entry consists of a mode string, a filename, a null byte, and a 20-byte SHA-1.
4. Extract and decode mode and filename, convert mode to integer.
5. Collect entries as tuples `(mode, path, sha1)`.

**Usage Example:**
```python
entries = read_tree(sha1='4b825dc642cb6eb9a060e54bf8d69288fbee4904')
for mode, path, sha1 in entries:
    print(f'{mode:o} {path} {sha1}')
```

---

### 4. `cat_file(mode, sha1_prefix)`

**Purpose:**  
Display contents or information about a Git object identified by SHA-1 prefix, in various modes (raw, size, type, pretty).

**Parameters:**  
- `mode` (str): Display mode. One of `'commit'`, `'tree'`, `'blob'`, `'size'`, `'type'`, or `'pretty'`.
- `sha1_prefix` (str): SHA-1 hash prefix of the object.

**Operation:**  
1. Read the object using `read_object`.
2. Depending on `mode`:
   - `'commit'`, `'tree'`, `'blob'`: output raw object data bytes to stdout.
   - `'size'`: print the size of the object data.
   - `'type'`: print the object type.
   - `'pretty'`: for commit/blob, output raw data; for tree, parse and print formatted tree entries.
3. Raise an error for unsupported modes or type mismatches.

**Usage Example:**
```python
cat_file('type', 'a1b2c3d4')  # Prints the object type
cat_file('pretty', 'a1b2c3d4')  # Prints prettified content of the object
```

---

### 5. `read_file(path)`

**Purpose:**  
Read the contents of a file from the filesystem as bytes.

**Parameters:**  
- `path` (str): Filesystem path to the file.

**Operation:**  
Open the file in binary mode and read all contents.

**Usage Example:**
```python
data = read_file('.git/objects/a1/b2c3d4...')
print(data[:20])  # Print first 20 bytes
```

---

## ASCII Diagram: Git Object Store Layout

```
.git/objects/
├── aa/
│   ├── bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb  # Object file (compressed)
│   ├── cccccccccccccccccccccccccccccccccccc
│   └── ...
├── bb/
│   └── aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
└── ...
```

- The first two characters of the SHA-1 hash form the directory.
- The remaining 38 characters form the filename.
- Objects are stored compressed in these files.

---

## Summary

The `object_reading.md` file documents the foundation for reading git objects from the object store. It explains how to locate objects by SHA-1 prefixes, decompress them, parse their headers to identify object types and sizes, and interpret tree objects into their constituent entries. These functions underpin many higher-level git commands such as `cat-file` for object inspection, enabling efficient retrieval and display of repository data. Understanding these basics is critical for working with git internals and implementing Git operations programmatically.