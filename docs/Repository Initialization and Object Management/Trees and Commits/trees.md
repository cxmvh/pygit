# trees.md

# Reading and Finding Tree Objects Recursively

---

## Overview

The `trees.md` file documents the functionality related to reading and finding Git tree objects recursively within the `pygit` project. It is part of the broader "Trees and Commits" section under "Repository Initialization and Object Management". Tree objects in Git represent directory structures, containing references to blobs (files) and other trees (subdirectories). This document covers how to read these tree objects, parse their entries, and recursively discover all objects referenced by a tree, which is essential for operations like commit exploration, object traversal, and pack creation.

---

## Functions

### `read_tree(sha1=None, data=None)`

Reads the contents of a Git tree object and returns a list of its entries.

**Purpose:**

- To parse a Git tree object either by its SHA-1 hash or by directly providing its raw data.
- Returns a list of tuples `(mode, path, sha1)` representing each entry in the tree.
- Tree entries include mode bits (indicating type and permissions), path (filename or directory), and SHA-1 hash of the referenced object.

**Parameters:**

- `sha1` (str, optional): Hex string SHA-1 of the tree object to read.
- `data` (bytes, optional): Raw bytes of the tree object. If `sha1` is provided, this is ignored.

**Preconditions:**

- At least one of `sha1` or `data` must be provided.
- If `sha1` is provided, the corresponding object must exist and be of type `'tree'`.

**How it works:**

1. If `sha1` is provided, calls `read_object(sha1)` to fetch the raw data of the tree.
2. If neither `sha1` nor `data` is provided, raises a `TypeError`.
3. Iterates over the raw data entries:
   - Each entry consists of a mode string, a path, a null byte delimiter, and a 20-byte SHA-1 digest.
   - Extracts mode and path by splitting on space.
   - Converts mode from octal string to integer.
   - Extracts 20-byte object SHA-1 and converts to hex string.
4. Returns a list of `(mode, path, sha1)` tuples.

**Example Usage:**

```python
# Read tree object by SHA-1 hash
entries = read_tree(sha1='a1b2c3d4e5f6...')
for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")

# Or, read from raw data bytes
raw_data = ...  # bytes obtained from some source
entries = read_tree(data=raw_data)
```

---

### `find_tree_objects(tree_sha1)`

Recursively finds all object hashes referenced within a tree and its subtrees.

**Purpose:**

- To discover all objects (trees and blobs) that are reachable from a given tree object.
- Returns a set of SHA-1 hashes including the tree itself and all its descendant objects.

**Parameters:**

- `tree_sha1` (str): SHA-1 hex string of the root tree object.

**How it works:**

1. Initializes a set with the root `tree_sha1`.
2. Reads the tree entries using `read_tree(tree_sha1)`.
3. For each entry:
   - Checks if the mode indicates a directory (using `stat.S_ISDIR(mode)`).
   - If directory, recursively calls `find_tree_objects` on that subtree SHA-1 and unions the results.
   - Otherwise, adds the blob SHA-1 to the set.
4. Returns the complete set of SHA-1s.

**Example Usage:**

```python
all_objects = find_tree_objects('a1b2c3d4e5f6...')
print(f"All objects reachable from tree: {all_objects}")
```

---

### Integration with `cat_file` (for Pretty Printing Trees)

The `cat_file` function supports a `'pretty'` mode to display tree contents in a human-readable form, leveraging `read_tree()` internally.

**Example Output Format:**

```
100644 blob 3b18e9f9f9a7f89e6eec1c0ea5c9d5f7c7a7b4e7    README.md
040000 tree 7c3f3b5a6d8b4f1234567890abcdef1234567890    src
```

Here, the mode is displayed octally, followed by the object type (`blob` or `tree`), the SHA-1, and the path.

---

## ASCII Diagram: Tree Object Structure

```
+--------------------------------------------+
| Tree Object Raw Data                        |
|                                            |
|  Entry 1:                                  |
|  +----------+------+---------------------+ |
|  | mode     | path | 20-byte SHA-1 hash  | |
|  +----------+------+---------------------+ |
|                                            |
|  Entry 2:                                  |
|  +----------+------+---------------------+ |
|  | mode     | path | 20-byte SHA-1 hash  | |
|  +----------+------+---------------------+ |
|                                            |
|  ...                                       |
+--------------------------------------------+

Each entry is encoded as:
"<mode> <path>\0<20-byte SHA-1>"
```

---

## Related Functions (Brief)

- `read_object(sha1_prefix)`  
  Reads an object of any type by SHA-1 prefix, returning `(object_type, data_bytes)`.

- `find_commit_objects(commit_sha1)`  
  Recursively finds all objects referenced by a commit, including trees and parents.

- `cat_file(mode, sha1_prefix)`  
  Displays object contents in various modes, including pretty-printing tree entries.

---

# Summary

This document provides comprehensive details on the recursive reading and exploration of Git tree objects in the `pygit` project. Using `read_tree` and `find_tree_objects`, users and developers can parse tree structures, enumerate all associated objects, and integrate these capabilities into higher-level commands like `cat_file` or commit traversal.

---

# Appendix: Key Code Snippets

```python
def read_tree(sha1=None, data=None):
    """Read tree object with given SHA-1 (hex string) or data, and return list
    of (mode, path, sha1) tuples.
    """
    if sha1 is not None:
        obj_type, data = read_object(sha1)
        assert obj_type == 'tree'
    elif data is None:
        raise TypeError('must specify "sha1" or "data"')
    i = 0
    entries = []
    for _ in range(1000):
        end = data.find(b'\x00', i)
        if end == -1:
            break
        mode_str, path = data[i:end].decode().split()
        mode = int(mode_str, 8)
        digest = data[end + 1:end + 21]
        entries.append((mode, path, digest.hex()))
        i = end + 1 + 20
    return entries


def find_tree_objects(tree_sha1):
    """Return set of SHA-1 hashes of all objects in this tree (recursively),
    including the hash of the tree itself.
    """
    objects = {tree_sha1}
    for mode, path, sha1 in read_tree(sha1=tree_sha1):
        if stat.S_ISDIR(mode):
            objects.update(find_tree_objects(sha1))
        else:
            objects.add(sha1)
    return objects
```

---

End of `trees.md` documentation.