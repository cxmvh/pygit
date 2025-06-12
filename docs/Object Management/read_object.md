# read_object.md

# Reading Raw Git Objects from Storage

## Overview

This document details the functionality for reading raw Git objects directly from the Git object store. It explains how objects are located, decompressed, and parsed to extract their type and raw data content. This capability is fundamental for many Git operations such as displaying object contents (`cat_file`), traversing trees, and verifying object integrity. The file fits into the broader "Object Management" section of the documentation tree, which covers functions for reading, writing, and manipulating Git objects including commits, trees, and blobs.

Understanding how to read raw Git objects is essential for implementing commands that inspect or manipulate repository history and structure, such as `git cat-file`, `git ls-files`, and other internal mechanisms that rely on accessing stored objects by their SHA-1 hashes.

---

## Important Functions

### `read_object(sha1_prefix)`

#### Purpose

Reads a Git object identified by a SHA-1 prefix from the Git object store and returns a tuple containing the object type and the raw data bytes. This function raises a `ValueError` if the object cannot be found or if the prefix is ambiguous.

#### Parameters

- `sha1_prefix` (`str`): A hexadecimal string prefix of the SHA-1 hash identifying the Git object. Must be at least 2 characters long.

#### How it works

1. Calls `find_object(sha1_prefix)` to locate the object file path in the `.git/objects` directory based on the prefix.
2. Reads the compressed object file content using `read_file(path)`.
3. Decompresses the content with `zlib.decompress`.
4. Parses the decompressed data, which consists of a header and the object data:
   - The header ends at the first null byte (`\x00`).
   - The header is of the format: `{object_type} {size}`, e.g., `blob 14`.
   - The object type (e.g., `blob`, `tree`, `commit`) and the size are extracted.
5. Verifies that the size in the header matches the actual data length.
6. Returns a tuple `(obj_type, data_bytes)`.

#### Example Usage

```python
obj_type, data = read_object('a1b2c3')
print('Object type:', obj_type)  # e.g., 'blob'
print('Data bytes:', data)       # raw bytes of the object
```

---

### `find_object(sha1_prefix)`

#### Purpose

Locate the file path of a Git object based on its SHA-1 prefix within the Git object store. Ensures that the prefix uniquely identifies a single object.

#### Parameters

- `sha1_prefix` (`str`): A hexadecimal string prefix of the SHA-1 hash (minimum length 2).

#### How it works

1. Validates that the prefix is at least 2 characters.
2. Determines the object directory as `.git/objects/<first two chars>`.
3. Lists all files in this directory that start with the remainder of the prefix.
4. Raises a `ValueError` if no objects or multiple objects match the prefix.
5. Returns the full file path to the unique matching object file.

#### Example Usage

```python
path = find_object('a1b2c3')
print('Object file path:', path)
# Output: .git/objects/a1/b2c3... (full path)
```

---

### `read_file(path)`

#### Purpose

Reads the entire contents of a file as bytes.

#### Parameters

- `path` (`str`): The file system path to the file.

#### How it works

- Opens the file in binary mode.
- Reads all bytes.
- Returns the bytes data.

#### Example Usage

```python
data = read_file('.git/objects/a1/b2c3de...')
print(data)  # raw bytes of the object file (compressed)
```

---

### `cat_file(mode, sha1_prefix)`

#### Purpose

Prints the contents or information about a Git object identified by a SHA-1 prefix to standard output. Supports multiple modes:

- `'commit'`, `'tree'`, `'blob'`: prints raw data bytes if object type matches.
- `'size'`: prints the size of the object data.
- `'type'`: prints the object type.
- `'pretty'`: prints a prettified version of the object (e.g., formatted tree entries).

#### Parameters

- `mode` (`str`): The mode of output.
- `sha1_prefix` (`str`): SHA-1 hash prefix of the object.

#### How it works

1. Calls `read_object(sha1_prefix)` to get `(obj_type, data)`.
2. Based on `mode`:
   - If mode is one of `'commit'`, `'tree'`, or `'blob'`, checks that the object's type matches and writes raw data to stdout.
   - If `'size'`, prints the length of the data.
   - If `'type'`, prints the object type.
   - If `'pretty'`:
     - For `'commit'` and `'blob'`, prints raw data.
     - For `'tree'`, parses tree entries and prints them in a readable format (mode, type, SHA-1, path).
3. Raises `ValueError` for unsupported modes or unexpected object types.

#### Example Usage

```python
cat_file('type', 'a1b2c3')   # prints: blob
cat_file('size', 'a1b2c3')   # prints: 1024
cat_file('pretty', 'a1b2c3') # prints prettified tree or commit info
```

---

### `read_tree(sha1=None, data=None)`

#### Purpose

Parses a Git tree object from either a SHA-1 or raw data bytes and returns a list of tuples describing the tree entries.

#### Parameters

- `sha1` (`str`, optional): SHA-1 hash of the tree object.
- `data` (`bytes`, optional): Raw data of the tree object.

At least one of `sha1` or `data` must be provided.

#### How it works

1. If `sha1` is given, reads the object and ensures it is a `tree`.
2. Parses the tree entries from raw data:
   - Each entry has the format: `<mode> <path>\0<20-byte SHA-1>`.
   - Iterates through the data, extracting mode, path, and SHA-1 of each entry.
3. Returns a list of tuples `(mode, path, sha1)`.

#### Example Usage

```python
entries = read_tree(sha1='d670460b4b4aece5915caf5c68d12f560a9fe3e4')
for mode, path, sha in entries:
    print(f"{mode:o} {path} {sha}")
```

---

## Additional Notes

### Object Storage Layout

Git objects are stored in the `.git/objects` directory structured as follows:

```
.git/objects/
    ├── aa/
    │    ├── bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
    │    ├── ...
    ├── bb/
    │    ├── cccccccccccccccccccccccccccccccccccc
    │    ├── ...
    └── ...
```

- The first two hex characters of the SHA-1 form the directory name.
- The remaining 38 hex characters form the filename.
- Objects are stored compressed using zlib.

### Object File Format

The decompressed object file content has this structure:

```
{object_type} {size}\0{data}
```

- `{object_type}`: string such as `blob`, `tree`, `commit`.
- `{size}`: decimal size of the data bytes.
- `\0`: null byte separator.
- `{data}`: raw object data.

### ASCII Diagram: Object Storage and Reading Flow

```
+----------------------------+
| Git Object SHA-1: a1b2c3...|
+----------------------------+
             |
             v
+----------------------------+
| .git/objects/a1/b2c3...    |  <-- compressed object file
+----------------------------+
             |
             v (read_file & decompress)
+----------------------------+
| "blob 14\0Hello, Git blob!"|  <-- decompressed content
+----------------------------+
             |
             v (parse header and data)
+----------------------------+
| obj_type = "blob"           |
| size = 14                  |
| data = b'Hello, Git blob!' |
+----------------------------+
```

---

# Summary

The `read_object.md` document covers the mechanisms to locate and read raw Git objects from the Git object store, returning their type and data. This forms the basis for higher-level commands and functions that manipulate Git repository contents and history.

By understanding these core functions (`read_object`, `find_object`, and `cat_file`), developers can extend or integrate Git object handling into custom tools or scripts.