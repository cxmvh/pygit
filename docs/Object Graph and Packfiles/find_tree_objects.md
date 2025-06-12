# find_tree_objects.md

## Overview

This document describes the functionality for finding all Git objects reachable from a given tree object. In Git, a tree object represents a directory snapshot and references blobs (files) and other trees (subdirectories). The ability to recursively find all objects reachable from a tree is crucial for operations such as packfile creation, repository integrity checks, and understanding the object graph.

This file fits into the broader **Object Graph and Packfiles** section of the documentation tree, which deals with managing commit graphs, sets of objects, and packfiles. It complements related documentation on finding commit objects, missing objects, and encoding objects for packfiles.

---

## Function Documentation

### `find_tree_objects(tree_sha1)`

#### Purpose

Recursively find and return all Git object SHA-1 hashes reachable from the specified tree object, including the tree itself, all blobs (files), and nested trees (subdirectories).

This function is essential for traversing the directory structure represented by a tree object and gathering the complete set of contained objects. It supports operations like packing objects for transfer, verifying repository completeness, and understanding repository state.

#### Parameters

- `tree_sha1` (str): The SHA-1 hash (hex string) identifying the tree object to examine.

#### Returns

- `set`: A set of SHA-1 hex strings representing all objects reachable from the tree, including the tree itself, all blobs, and all nested trees recursively.

#### Operation Steps

1. Initialize a set `objects` containing the SHA-1 of the starting tree.
2. Use `read_tree(tree_sha1)` to parse the tree object into (mode, path, sha1) tuples.
3. For each entry in the tree:
   - If the mode indicates a directory (subtree), recursively call `find_tree_objects` on the subtree's SHA-1 and add all returned objects to the set.
   - Otherwise (a blob), add the blob's SHA-1 to the set.
4. Return the complete set of collected SHA-1 hashes.

#### Example Usage

```python
tree_hash = '4a9f3d7e2a3b6f6b...'

# Find all objects reachable from the tree
all_objects = find_tree_objects(tree_hash)

print("Objects reachable from tree {}:".format(tree_hash))
for obj_hash in sorted(all_objects):
    print(obj_hash)
```

This example prints all SHA-1 hashes of objects (blobs and nested trees) reachable from the specified tree.

---

## Related Concepts

### Tree Object Structure

A tree object contains entries representing files and directories in a directory snapshot. Each entry has:

- **mode**: File mode (e.g., regular file, directory).
- **path**: File or directory name.
- **sha1**: SHA-1 hash of the referenced blob or tree.

```
+-------------------------------+
| Tree Object (SHA-1: tree_sha1)|
+-------------------------------+
| mode path sha1                |----> blob (file)
| mode path sha1                |----> tree (subdirectory)
| ...                          |
+-------------------------------+
```

### Recursive Traversal

The algorithm uses recursion to explore nested trees:

```
find_tree_objects(tree_sha1)
   |
   |---> For each entry in tree:
           |-- if entry is tree: find_tree_objects(entry.sha1)
           |-- else: add blob sha1
```

---

## ASCII Diagram Illustrating Recursive Tree Object Traversal

```
Tree (A)
├── file1.txt (blob B)
├── file2.txt (blob C)
└── subdir (tree D)
     ├── file3.txt (blob E)
     └── nested (tree F)
          └── file4.txt (blob G)

find_tree_objects(A) returns:
{A, B, C, D, E, F, G}
```

- `A` references blobs `B` and `C` and tree `D`.
- `D` references blob `E` and tree `F`.
- `F` references blob `G`.

The function collects all these objects recursively.

---

## Full Function Code Reference

```python
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

## See Also

- [`read_tree(sha1=None, data=None)`](read_tree.md) — Parses tree objects and returns their entries.
- [`find_commit_objects(commit_sha1)`](find_commit_objects.md) — Finds all objects reachable from a commit, including its tree and parent commits.
- [`create_pack(objects)`](create_pack.md) — Creates a Git packfile from a set of objects.
- [`hash_object(data, obj_type, write=True)`](object_storage.md) — Hashes and optionally stores Git objects.

---

This documentation provides a clear understanding of how to identify all Git objects reachable from a tree by recursively traversing its entries, which is fundamental for Git's object management and transfer processes.