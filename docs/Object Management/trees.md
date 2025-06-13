# trees.md

## Overview

This document provides detailed information on reading and finding Git tree objects recursively within the pygit project. Tree objects in Git represent directory contents at a given commit, containing references to blobs (files) and other trees (subdirectories). Understanding how to parse and traverse these tree objects is essential for operations such as displaying repository contents, constructing commits, and managing object relationships. This file fits under the "Object Management" section of the documentation, complementing other files that describe git object handling like blobs, commits, and pack files. The core functionality relies on reading tree objects from the object store, parsing their entries, and recursively traversing subtrees to gather all contained objects.

---

## Function Documentation

### `read_tree(sha1=None, data=None)`

#### Purpose
Reads a Git tree object either by its SHA-1 hash or directly from provided raw data. It returns a list of entries contained in that tree, where each entry describes a file or directory with its mode, path, and SHA-1 hash.

#### Parameters
- `sha1` (`str`, optional): The SHA-1 hex string of the tree object to read.
- `data` (`bytes`, optional): Raw bytes of the tree object. Either `sha1` or `data` must be provided.

#### Preconditions
- Exactly one of `sha1` or `data` must be specified.
- If `sha1` is provided, the object must exist in the object store and be of type `tree`.

#### Operation
1. If `sha1` is provided:
   - Calls `read_object(sha1)` to retrieve the object type and raw data.
   - Asserts that the object type is `tree`.
2. If `data` is not provided and `sha1` is `None`, raises an error.
3. Parses the raw tree data sequentially:
   - Each entry consists of an ASCII mode and path string terminated by a NUL byte (`\x00`).
   - After the NUL byte, the next 20 bytes represent the SHA-1 hash of the referenced object.
4. Extracts mode (as an octal integer), path (string), and SHA-1 (hex string) for each entry.
5. Returns a list of tuples `(mode, path, sha1)` for all entries in the tree.

#### Example Usage

```python
# Read tree by SHA-1
entries = read_tree(sha1='a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0')
for mode, path, sha1 in entries:
    print(f'Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}')

# Alternatively, read tree from raw data bytes
raw_data = b'100644 README.md\x00' + bytes.fromhex('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')
entries_from_data = read_tree(data=raw_data)
```

---

### `find_tree_objects(tree_sha1)`

#### Purpose
Recursively finds all Git objects referenced by a tree object, including blobs (files) and subtrees (directories), returning a complete set of SHA-1 hashes. This is critical for operations that need the full object graph of a commit, such as packing objects for transfer or integrity checks.

#### Parameters
- `tree_sha1` (`str`): SHA-1 hex string of the tree object to explore.

#### Preconditions
- The specified SHA-1 must correspond to a valid tree object.

#### Operation
1. Initialize a set with the given tree SHA-1.
2. Read the tree entries using `read_tree(tree_sha1)`.
3. For each entry:
   - If the mode indicates a directory (`stat.S_ISDIR(mode)`), recursively call `find_tree_objects` on that subtree SHA-1 and update the set.
   - Otherwise, add the blob SHA-1 to the set.
4. Return the aggregated set of all object SHA-1s found.

#### Example Usage

```python
tree_sha1 = 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s9t0'
all_objects = find_tree_objects(tree_sha1)
print(f'Objects in tree {tree_sha1}:')
for obj_sha1 in sorted(all_objects):
    print(obj_sha1)
```

#### ASCII Diagram: Recursive Tree Traversal

```
Tree (sha1: T1)
├── file1.txt (blob: B1)
├── file2.txt (blob: B2)
└── subdir (tree: T2)
    ├── file3.txt (blob: B3)
    └── nested (tree: T3)
        └── file4.txt (blob: B4)

find_tree_objects(T1) returns {T1, B1, B2, T2, B3, T3, B4}
```

---

### `cat_file(mode, sha1_prefix)`

#### Purpose
Outputs the contents or metadata of a Git object identified by a SHA-1 prefix, supporting multiple modes such as printing raw data, size, type, or a prettified view of the object. This function is often used for inspecting objects in the object store.

#### Parameters
- `mode` (`str`): Specifies the output mode. Supported values:
  - `'commit'`, `'tree'`, `'blob'`: Print raw object data bytes.
  - `'size'`: Print size of the object in bytes.
  - `'type'`: Print object type as string.
  - `'pretty'`: Print a human-friendly representation, including formatted tree entries.
- `sha1_prefix` (`str`): SHA-1 prefix string identifying the object.

#### Preconditions
- The object corresponding to `sha1_prefix` must exist.
- If mode is `'commit'`, `'tree'`, or `'blob'`, the actual object type must match the requested mode.

#### Operation
1. Calls `read_object(sha1_prefix)` to get `(obj_type, data)`.
2. Depending on `mode`:
   - For `'commit'`, `'tree'`, `'blob'`: verify type matches and write raw data to stdout.
   - For `'size'`: print length of `data`.
   - For `'type'`: print `obj_type`.
   - For `'pretty'`:  
     - If `commit` or `blob`, write raw data.
     - If `tree`, parse entries using `read_tree(data=data)` and print each entry with mode, type, SHA-1, and path.
3. Raises errors for unknown modes or mismatches.

#### Example Usage

```python
# Print pretty tree object
cat_file('pretty', 'a1b2c3d4')

# Print size of blob object
cat_file('size', 'e5f6g7h8')
```

---

### Supporting Functions (Brief Summary)

- `read_object(sha1_prefix)`: Reads raw object data and returns `(type, data)`.
- `find_object(sha1_prefix)`: Locates object file path in `.git/objects`.
- `read_file(path)`: Reads and returns file bytes.
- `hash_object(data, obj_type, write=True)`: Hashes and optionally writes object to storage.

---

## Summary Diagram: Object Reading Flow for Trees

```
+----------------+
| cat_file()     |
+--------+-------+
         |
         v
+----------------+    calls    +----------------+
| read_object()  +-----------> | find_object()  |
+--------+-------+             +--------+-------+
         |                              |
         |                              v
         |                      +----------------+
         |                      | read_file(path)|
         |                      +----------------+
         |
         v
+----------------+ parses +----------------+
| raw object data |<-------| decompress()   |
+--------+-------+        +----------------+
         |
         v
+----------------+
| read_tree()    |<---- parses tree entries
+----------------+
```

---

# End of `trees.md` Documentation File Content