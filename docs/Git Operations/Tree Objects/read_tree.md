# read_tree.md

# Reading Tree Object Contents and Entries

---

## Overview

The `read_tree.md` file documents the functionality for reading Git tree objects within the `pygit` project. Tree objects in Git represent directory contents at a specific commit, storing references to blobs (files) and other trees (subdirectories) along with their modes and paths. This file's documentation explains how to parse these tree objects, either by their SHA-1 hash or raw data, and extract detailed entries of their contents.

Situated within the **Object Handling** section of the overall documentation, this file complements other files that handle Git objects such as blobs and commits. Specifically, it details the `read_tree` function, which is crucial for traversing the directory structure represented in a Git repository's history. This function enables higher-level operations like recursively finding all objects referenced by a tree or displaying tree contents.

---

## Function: `read_tree`

### Purpose

`read_tree` reads and parses a Git tree object, which represents the hierarchical structure of files and directories at a given point in the repository's history. The function returns a list of entries describing the contents of the tree, including file modes, file or directory paths, and SHA-1 hashes of the referenced objects.

### Signature

```python
def read_tree(sha1=None, data=None):
```

### Parameters

- `sha1` (str, optional): The SHA-1 hash (hex string) of the tree object to read. If provided, the function reads the object data from the Git object database.
- `data` (bytes, optional): Raw bytes data of a tree object. If provided, the function parses this data directly.
  
At least one of `sha1` or `data` must be specified; otherwise, a `TypeError` is raised.

### Returns

- A list of tuples `(mode, path, sha1)`:
  - `mode` (int): The file mode in octal, indicating the type and permissions of the entry.
  - `path` (str): The file or directory path relative to this tree.
  - `sha1` (str): The SHA-1 hash (hex string) of the referenced object (blob or subtree).

### Description and Operation

1. **Input Validation**:
   - If `sha1` is provided, the function uses `read_object(sha1)` to retrieve the object type and data.
   - It asserts that the object type is `'tree'`.
   - If `data` is provided directly, it uses that for parsing.
   - If neither is provided, it raises a `TypeError`.

2. **Parsing the Tree Data**:
   - The tree object format consists of multiple entries concatenated together.
   - Each entry is structured as:
     - `<mode> <path>\0<20-byte SHA-1 binary digest>`
   - The function parses each entry by searching for the null byte (`\0`) delimiter to separate the mode/path string from the binary SHA-1.
   - The mode and path are decoded from the bytes before the null byte.
   - The mode is parsed as an octal integer.
   - The 20 bytes following the null byte represent the SHA-1 hash in binary form; this is converted to a hex string.
   - The function accumulates these tuples into a list.

3. **Return Value**:
   - Returns the list of `(mode, path, sha1)` tuples representing the tree entries.

### Example Usage

```python
# Read a tree object by SHA-1 hash
tree_sha1 = '3a5b8e2c1f9d6b4e7f8c9d0a1b2c3d4e5f67890a'
entries = read_tree(sha1=tree_sha1)

for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

Output might look like:

```
Mode: 0o100644, Path: README.md, SHA-1: a1b2c3d4e5f67890abcdef1234567890abcdef12
Mode: 0o40000, Path: src, SHA-1: b2c3d4e5f67890abcdef1234567890abcdef1234
...
```

Alternatively, if you have raw tree data:

```python
raw_tree_data = ...  # bytes of a tree object
entries = read_tree(data=raw_tree_data)
```

---

## Supporting Functions

To understand and use `read_tree` effectively, it is helpful to know about related functions:

### `read_object(sha1_prefix)`

Reads a Git object by SHA-1 prefix and returns a tuple `(object_type, data_bytes)`.

Example:

```python
obj_type, data = read_object('3a5b8e2c1f9d6b4e7f8c9d0a1b2c3d4e5f67890a')
print(obj_type)  # 'tree'
```

### `find_object(sha1_prefix)`

Finds the path to the object file in the Git object database corresponding to the given SHA-1 prefix.

### `read_file(path)`

Reads the raw bytes content of a file at the specified path.

---

## ASCII Diagram: Tree Object Structure

```
+------------------------------------------------+
| Tree Object Data (bytes)                        |
|                                                |
| [Entry 1] [Entry 2] [Entry 3] ... [Entry N]    |
+------------------------------------------------+

Each Entry:
+------------------------------+
| Mode (ascii octal) + ' '     |  e.g., "100644 "
| Path (ascii)                 |  e.g., "README.md"
| Null byte (\0)               |
| 20-byte SHA-1 binary digest  |  raw binary SHA-1 hash of blob/tree
+------------------------------+

Example Entry (hex view):
"31303036343420524541444d452e6d6400" + (20 bytes SHA-1)
 ^^^^^^^^  ^^^^^^^^^^^  ^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 mode     path        null sha1 binary
```

---

## Summary

The `read_tree` function is essential for interpreting Git tree objects, enabling the `pygit` system to understand the directory contents stored within a commit. By converting the raw tree data into structured entries, this function allows other components to navigate, display, and manipulate the repository's file hierarchy at any commit point.