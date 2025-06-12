# tree_objects.md

---

## Overview

This document covers working with Git tree objects in the `pygit` project, focusing on reading tree objects and recursively finding all objects referenced within trees. Tree objects in Git represent directories and contain references to blobs (files) and other trees (subdirectories). Understanding how to manipulate tree objects is essential for interpreting the repository structure, traversing the commit history, and performing operations such as displaying repository contents or preparing objects for transfer.

`tree_objects.md` fits within the broader "Git Object Handling" section, complementing other documents that describe reading, hashing, and writing Git objects. It specifically addresses:

- Reading tree objects by SHA-1 or raw data.
- Parsing tree entries into their components: mode, path, and SHA-1.
- Recursively finding all objects contained within a tree and its subtrees.

---

## Function Documentation

### `read_tree(sha1=None, data=None)`

#### Purpose

Reads a Git tree object either by its SHA-1 hash or from raw data bytes and returns a list of entries contained in the tree. Each entry represents a file or subdirectory within the tree, described by a tuple `(mode, path, sha1)`.

This function is fundamental for interpreting the contents of a tree object, allowing higher-level functions to traverse the directory structure stored in Git.

#### Parameters

- `sha1` (str, optional): SHA-1 hash (hex string) of the tree object to read. If provided, the function reads and decompresses the object from the Git object store.
- `data` (bytes, optional): Raw bytes of a tree object. If `sha1` is not provided, `data` must be given.

At least one of `sha1` or `data` must be provided.

#### Preconditions

- If `sha1` is provided, the corresponding object must exist and be of type `'tree'`.
- If neither `sha1` nor `data` is provided, the function raises `TypeError`.

#### How It Works

1. If `sha1` is specified, the function calls `read_object(sha1)` to retrieve the object type and raw data, asserting that the object is a tree.
2. If `data` is provided, it uses the data directly.
3. It initializes an index `i` at 0 and an empty list `entries`.
4. It loops (up to a large limit to avoid infinite loops) to parse each tree entry:
   - Finds the null byte (`\x00`) that separates the mode/path section from the SHA-1 digest.
   - Extracts and decodes the mode and path from the bytes before the null byte.
   - Converts the mode string from octal to an integer.
   - Extracts the 20-byte SHA-1 digest following the null byte.
   - Converts the digest to a hex string.
   - Appends a tuple `(mode, path, sha1)` to the entries list.
   - Advances the index to the start of the next entry.
5. Returns the list of `(mode, path, sha1)` tuples.

#### Example Usage

```python
# Read tree entries from a tree object identified by its SHA-1
tree_sha1 = "4b825dc642cb6eb9a060e54bf8d69288fbee4904"
entries = read_tree(sha1=tree_sha1)

for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

---

### `find_tree_objects(tree_sha1)`

#### Purpose

Recursively finds all Git object SHA-1 hashes contained in the given tree object, including:

- The SHA-1 of the tree itself.
- SHA-1s of all blobs (files) referenced by the tree.
- SHA-1s of all subtree objects (directories) and their contained objects.

This function is crucial for operations that require enumerating all objects reachable from a particular tree, such as packing objects for transfer or verifying repository integrity.

#### Parameters

- `tree_sha1` (str): SHA-1 hash (hex string) of the tree object to analyze.

#### Preconditions

- The provided `tree_sha1` must correspond to a valid tree object in the Git object store.

#### How It Works

1. Initializes a set `objects` with the SHA-1 of the current tree.
2. Calls `read_tree(sha1=tree_sha1)` to get all entries in the tree.
3. Iterates over each `(mode, path, sha1)` tuple in the tree entries:
   - Checks if the mode indicates a directory (using `stat.S_ISDIR(mode)`).
   - If it is a directory, recursively calls `find_tree_objects(sha1)` and adds all returned objects to the set.
   - Otherwise, adds the blob's SHA-1 to the set.
4. Returns the full set of SHA-1 hashes found.

#### Example Usage

```python
# Find all objects reachable from a tree object
root_tree_sha1 = "4b825dc642cb6eb9a060e54bf8d69288fbee4904"
all_objects = find_tree_objects(root_tree_sha1)

print(f"Objects contained in tree {root_tree_sha1}:")
for obj_sha1 in sorted(all_objects):
    print(obj_sha1)
```

---

## ASCII Diagram: Git Tree Object Structure

```
Git Tree Object (raw data bytes)

+---------------------+
| mode path \x00 sha1 | entry 1 (file or subtree)
+---------------------+
| mode path \x00 sha1 | entry 2
+---------------------+
|         ...         |
+---------------------+

Each entry:
- mode: file mode in octal (e.g., 100644 for regular file, 40000 for directory)
- path: filename or directory name as UTF-8 string
- \x00: null byte separator
- sha1: 20 bytes binary SHA-1 hash of the object referenced
```

---

## Relationship to Other Functions

- `read_tree` depends on `read_object` to retrieve and decompress the tree object by SHA-1.
- `find_tree_objects` uses `read_tree` to recursively enumerate all nested objects.
- These functions are used by higher-level operations such as `find_commit_objects` which find all objects reachable from a commit's root tree.
- `cat_file` uses `read_tree` to pretty-print tree contents when displaying objects.

---

# Summary

The functions in this document provide essential capabilities to work with Git tree objects:

- `read_tree` parses tree objects into their constituent entries.
- `find_tree_objects` recursively collects all objects referenced by a tree, facilitating operations like packing and repository traversal.

These capabilities enable `pygit` to interpret and manipulate the directory structure stored in Git repositories, supporting core commands such as `cat-file`, `commit`, and `push`.