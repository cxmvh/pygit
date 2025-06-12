# read_tree.md

# Parsing Tree Objects into Their Entries

---

## Overview

The `read_tree.md` document provides technical reference for parsing Git tree objects into their constituent entries. A tree object in Git represents a directory listing, containing references to blobs (files) or other trees (subdirectories), along with associated metadata such as file modes and paths. This file explains how tree objects are read and decoded into usable data structures, enabling other operations such as diffing, checkout, and commit creation.

Within the broader repository documentation, this file fits under the **Object Handling and Git Objects** section, complementing related files like `write_tree.md` (which covers the inverse operation of writing tree objects) and `read_object.md`. It is also related to diffing operations, as trees define directory snapshots that can be compared.

Understanding tree parsing is essential for implementing Git functionality that involves inspecting or traversing repository trees, such as displaying file lists at a commit, comparing directory states, or preparing commits.

---

## Function Documentation

### `read_tree(sha1=None, data=None)`

#### Purpose

Reads a Git tree object and parses it into a list of its entries. Each entry corresponds to a file or subdirectory within the tree and includes its mode (permissions and type), path (filename), and SHA-1 hash (object ID). This function accepts either:

- `sha1`: A SHA-1 hex string identifying the tree object to read from the object store, or
- `data`: Raw bytes of the tree object directly.

Exactly one of `sha1` or `data` must be provided.

#### Parameters

- `sha1` (str, optional): The SHA-1 hex identifier of the tree object to read.
- `data` (bytes, optional): Raw byte content of a tree object.

#### Returns

- `List[Tuple[int, str, str]]`: A list of tuples, each containing `(mode, path, sha1)` where:
  - `mode` is an integer file mode (e.g., 100644 for a regular file),
  - `path` is the file or directory name as a string,
  - `sha1` is the hex string of the referenced object.

#### Preconditions

- If `sha1` is provided, the referenced object must be a tree.
- If neither `sha1` nor `data` is provided, a `TypeError` is raised.

#### Operation Details

1. If `sha1` is specified:
   - Use `read_object(sha1)` to load the object data and verify it is a tree.
2. If `data` is specified directly, use it as the raw tree data.
3. Parse the raw tree data sequentially:
   - Each tree entry is encoded as: `<mode> <path>\0<20-byte SHA-1>`
   - The mode and path are ASCII strings terminated by a null byte (`\x00`).
   - The 20 bytes following the null byte represent the binary SHA-1 hash.
4. Extract each entry, convert mode from octal string to int, decode path as UTF-8, and convert the SHA-1 bytes to a hex string.
5. Continue until no more entries can be parsed.

#### Example Usage

```python
# Read tree entries by SHA-1
tree_sha1 = '3a4e8c7d1234567890abcdef1234567890abcdef'
entries = read_tree(sha1=tree_sha1)
for mode, path, sha1 in entries:
    print(f'Mode: {mode:o}, Path: {path}, SHA-1: {sha1}')
```

Or using raw data:

```python
_, raw_data = read_object('3a4e8c7d1234567890abcdef1234567890abcdef')
entries = read_tree(data=raw_data)
```

---

## ASCII Diagram: Tree Object Structure

```
+-----------------------------+
| Tree Object Raw Data         |
+-----------------------------+
| Entry 1:                    |
|  "100644 filename.txt" + \0  |
|  20-byte SHA-1 hash          |
+-----------------------------+
| Entry 2:                    |
|  "040000 subdir" + \0        |
|  20-byte SHA-1 hash          |
+-----------------------------+
| ...                         |
+-----------------------------+
```

Each entry concatenates the mode and path as ASCII text, followed by a null byte, then the binary SHA-1 hash of the referenced object.

---

## Related Functions Overview

- `read_object(sha1_prefix)`: Reads and decompresses any Git object given a SHA-1 prefix, returning its type and raw data bytes.
- `write_tree()`: Writes a new tree object from index entries, producing the raw tree data that `read_tree` parses.
- `cat_file()`: For displaying information about tree objects using `read_tree` internally to pretty-print contents.

---

## Summary

The `read_tree` function is fundamental for decoding the directory structure snapshots Git stores as tree objects. By converting binary tree data into a list of `(mode, path, sha1)` tuples, it enables higher-level operations to enumerate repository contents, traverse trees recursively, and compare directory states. Its close integration with object reading functions ensures consistent and reliable access to Git's internal data.