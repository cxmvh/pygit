# tree_handling.md

# Functions Related to Reading and Parsing Tree Objects

---

## Overview

This document describes the functions involved in reading and parsing Git tree objects within the `pygit` project. Tree objects in Git represent directory structures, mapping filenames to corresponding blob (file data) or subtree (subdirectory) objects. The functions detailed here facilitate accessing the contents of these tree objects, interpreting their raw data, and integrating with other components like the index and commit objects. This file fits under the broader "Object Handling" section of the documentation tree, particularly focusing on tree-specific operations. These functions support core Git operations such as displaying tree contents (`cat_file`), traversing the repository snapshot, and building tree objects from the index.

---

## Function Documentation

### 1. `read_tree(sha1=None, data=None)`

#### Purpose

Reads a Git tree object either by its SHA-1 hash or directly from raw object data bytes. It parses the tree content and returns a list of entries, where each entry is a tuple containing:

- `mode` (int): File mode bits (e.g., 100644 for normal files, 40000 for directories)
- `path` (str): Filename or directory name
- `sha1` (str): SHA-1 hash of the referenced object (blob or subtree)

This function is essential for interpreting the internal structure of Git tree objects.

#### Parameters

- `sha1` (str, optional): Hexadecimal SHA-1 string identifying the tree object to read.
- `data` (bytes, optional): Raw bytes of a tree object. Must be provided if `sha1` is not.

**Preconditions:**

- Either `sha1` or `data` must be specified, but not both.
- If `sha1` is specified, the referenced object must exist and be of type 'tree'.

#### Operation

1. If `sha1` is provided:
   - Calls `read_object(sha1)` to get the raw data of the tree object.
   - Asserts the object type is `'tree'`.
2. If `data` is provided directly, uses it as the tree data.
3. Parses the tree data sequentially:
   - Each entry consists of a mode string, a filename, a null byte separator, and a 20-byte SHA-1 digest.
   - Iterates until no more entries are found.
4. Converts mode string from octal to integer.
5. Extracts and converts the SHA-1 digest to a hex string.
6. Collects all parsed entries into a list.

#### Returns

- List of tuples: `[(mode, path, sha1), ...]`

#### Example Usage

```python
# Read tree entries by SHA-1
tree_sha1 = 'a1b2c3d4e5f6g7h8i9j0klmnopqrstuvwx123456'
entries = read_tree(sha1=tree_sha1)
for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")

# Alternatively, read tree entries from raw data bytes
obj_type, data = read_object(tree_sha1)
entries = read_tree(data=data)
```

#### ASCII Diagram: Tree Object Structure

```
+----------------------------+
| Tree Object Raw Data Bytes  |
+----------------------------+
| mode path\x00 sha1 (20 bytes) | mode path\x00 sha1 (20 bytes) | ... |
+----------------------------+

Each entry:
  - mode: ASCII octal string (e.g., "100644")
  - path: filename string
  - \x00: null separator
  - sha1: 20 raw bytes representing SHA-1 hash
```

---

### 2. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the object store by its SHA-1 prefix, decompresses it, and returns its type and raw data bytes.

#### Parameters

- `sha1_prefix` (str): A prefix of a SHA-1 hash identifying the object.

#### Operation

1. Calls `find_object(sha1_prefix)` to locate the object file in the `.git/objects` directory.
2. Reads and decompresses the object file using `zlib`.
3. Parses the object header to extract the object type and size.
4. Extracts the raw data bytes of the object.
5. Verifies the extracted size matches the header size.

#### Returns

- Tuple: `(obj_type, data_bytes)`

#### Example Usage

```python
obj_type, data = read_object('a1b2c3')
print(f"Object type: {obj_type}")
print(f"Data length: {len(data)} bytes")
```

---

### 3. `find_object(sha1_prefix)`

#### Purpose

Finds the full object file path in the Git object store given a SHA-1 prefix. Ensures the prefix uniquely identifies a single object.

#### Parameters

- `sha1_prefix` (str): At least first two characters of SHA-1 hash.

#### Operation

1. Validates `sha1_prefix` length ≥ 2.
2. Looks under `.git/objects/<first_two_chars>/` directory.
3. Lists files starting with the remaining prefix.
4. Raises errors if no or multiple matches found.
5. Returns full path to the object file.

#### Returns

- File path (str) of the object file.

#### Example Usage

```python
path = find_object('a1b2c3')
print(f"Object path: {path}")
```

---

### 4. `cat_file(mode, sha1_prefix)`

#### Purpose

Displays information or contents of a Git object based on the specified mode. Supports 'commit', 'tree', 'blob', 'size', 'type', and 'pretty' modes for object presentation.

#### Parameters

- `mode` (str): Determines output format ('commit', 'tree', 'blob', 'size', 'type', 'pretty').
- `sha1_prefix` (str): SHA-1 prefix of the object to display.

#### Operation

1. Reads the object using `read_object`.
2. Depending on `mode`:
   - For 'commit', 'tree', 'blob': outputs raw data bytes (asserting type match).
   - For 'size': prints object data size.
   - For 'type': prints object type.
   - For 'pretty':
     - For 'commit' and 'blob', outputs raw data.
     - For 'tree', parses tree entries and prints formatted lines with mode, type, SHA-1, and path.
3. Raises errors for unexpected modes or types.

#### Example Usage

```python
# Pretty print tree object
cat_file('pretty', 'a1b2c3')

# Print size of blob object
cat_file('size', 'd4e5f6')
```

#### ASCII Diagram: cat_file 'pretty' mode for tree objects

```
+---------------------+
| Tree entries output |
+---------------------+
| mode  type  sha1   path |
| 100644 blob  e69de... README.md |
| 40000  tree  4b825... src     |
+---------------------+
```

---

## Additional Functions Related to Tree Handling

While `read_tree` is the central function for parsing tree objects, it works closely with other functions:

- `read_object`: Reads raw Git objects.
- `find_object`: Locates object files by SHA-1 prefix.
- `cat_file`: Displays object contents, including trees in human-readable form.
- `find_tree_objects(tree_sha1)`: Recursively collects all object SHA-1s referenced by a tree and its subtrees.
- `write_tree()`: Writes a new tree object from the current index entries (not detailed here but relevant).

---

## Summary

This documentation provides a detailed overview of functions to read and parse tree objects in Git via the `pygit` implementation. Understanding tree object structure and how to access it is critical for many Git operations, including browsing repository contents, committing changes, and synchronizing objects. The `read_tree` function is the key utility for interpreting raw tree data, complemented by functions like `read_object` and `cat_file` for object retrieval and display.

---

# End of tree_handling.md documentation content.