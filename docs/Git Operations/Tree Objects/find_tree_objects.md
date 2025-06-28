# find_tree_objects.md

## Overview

This document describes the `find_tree_objects` function, which recursively finds all Git objects referenced by a tree object. In Git, a tree object represents a directory and contains references (SHA-1 hashes) to blobs (file contents) and other trees (subdirectories). This function is essential for traversing and collecting all objects that a given tree references, directly or indirectly.

Located within the "Tree and Commit Object Management" section of the pygit documentation, this file complements related functions such as `read_tree` and `find_commit_objects`. It plays a critical role in operations like pushing commits, where all related objects must be identified and sent to a remote repository.

---

## Function: find_tree_objects(tree_sha1)

### Purpose

`find_tree_objects` returns a set of SHA-1 hashes representing all Git objects referenced by the given tree object, including the tree itself. This includes:

- The tree object's own SHA-1.
- All blob objects (files) referenced by the tree.
- All subtrees (subdirectories) and their referenced objects recursively.

This recursive traversal is necessary because a tree can contain nested trees, forming a directory hierarchy.

### Parameters

- `tree_sha1` (str): The SHA-1 hash (hex string) of the tree object to inspect.

### Returns

- `set` of `str`: A set of SHA-1 hashes (hex strings) of all objects referenced by the tree, recursively.

### Preconditions

- The `tree_sha1` must refer to a valid tree object in the Git object database.
- The function depends on `read_tree`, which parses the tree object into entries.

### Operation Details

1. Initialize a set `objects` with the `tree_sha1` itself.
2. Call `read_tree` on the given SHA-1 to get a list of entries. Each entry is a tuple `(mode, path, sha1)` where:
   - `mode` is the file mode (e.g., directory or blob).
   - `path` is the filename or directory name.
   - `sha1` is the SHA-1 hash of the referenced object.
3. Iterate over each entry:
   - If the entry is a directory (`stat.S_ISDIR(mode)` is True), recursively call `find_tree_objects` on the subtree SHA-1 and add all returned objects to `objects`.
   - Otherwise, add the SHA-1 of the blob directly to `objects`.
4. Return the complete set of objects.

### Example Usage

```python
tree_sha1 = 'a3c1e9f9d7e8b2f9c7e7a4f8d8b9a0d1c2e3f4a5'
all_objects = find_tree_objects(tree_sha1)
print(f"Objects referenced by tree {tree_sha1}:")
for obj in sorted(all_objects):
    print(obj)
```

This example fetches all objects (including blobs and subtrees) referenced by the given tree SHA-1 and prints their hashes.

---

## Related Functions

- `read_tree(sha1)`: Parses a tree object and returns a list of `(mode, path, sha1)` tuples.
- `find_commit_objects(commit_sha1)`: Recursively finds all objects referenced by a commit, including its tree and parent commits.
- `read_object(sha1_prefix)`: Reads a Git object by SHA-1 prefix.
  
---

## ASCII Diagram: Recursive Tree Object References

```
Tree Object (SHA-1: T)
|
+-- Blob Object (file1.txt) ---- SHA-1: B1
|
+-- Blob Object (file2.txt) ---- SHA-1: B2
|
+-- Subtree (directory: src) ---- SHA-1: T_sub
     |
     +-- Blob Object (main.py) ---- SHA-1: B3
     |
     +-- Blob Object (utils.py) --- SHA-1: B4
```

- The function starts at tree `T`.
- It adds `T` itself to the result set.
- For each entry:
  - If it's a blob (file), add its SHA-1.
  - If it's a subtree (directory), recursively traverse and add all referenced objects.

---

## Code Snippet

```python
import stat

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

## Summary

The `find_tree_objects` function is a vital utility in pygit’s object management system. It enables recursive traversal of tree objects to collect all referenced Git objects, forming the foundational operation for commands like push and fetch that require identifying all objects related to a commit's tree. Understanding this function helps in grasping how Git represents directory structures internally and how pygit manages these objects efficiently.