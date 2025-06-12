# object_traversal.md

# Advanced Object Traversal: Recursive Object Finding Utilities for Commits and Trees

---

## Overview

This document describes the functions that enable recursive traversal and discovery of all Git objects (commits, trees, blobs) related to a given commit or tree SHA-1 hash. These utilities are essential for operations that need to gather the complete set of objects reachable from a commit, such as preparing objects for pushing or performing repository integrity checks.

Within the broader documentation tree, this file is placed under **Advanced Object Traversal**, a subsection of **Repository Initialization and Setup**. It complements other files focused on object storage, reading, and writing by providing recursive algorithms to enumerate all related objects in the commit graph and tree hierarchy.

---

## Function Documentation

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds and returns a set of all SHA-1 hashes of objects contained within a tree object, including all nested subtrees and blob entries, along with the tree itself.

**Parameters:**  
- `tree_sha1` (`str`): The SHA-1 hash (hex string) of the tree object to traverse.

**Returns:**  
- `set[str]`: A set of SHA-1 hex strings representing all objects reachable from the given tree.

**Operation:**  
1. Initialize a set containing the current tree's SHA-1.  
2. Read the entries of the tree using `read_tree(tree_sha1)`, which yields tuples of `(mode, path, sha1)`.  
3. For each entry:  
    - If the mode indicates a directory (subtree), recursively call `find_tree_objects` on its SHA-1 and add all resulting objects to the set.  
    - Otherwise, add the blob's SHA-1 to the set.  
4. Return the accumulated set of objects.

**Example Usage:**

```python
tree_sha1 = "e69de29bb2d1d6434b8b29ae775ad8c2e48c5391"
all_tree_objects = find_tree_objects(tree_sha1)
print(f"Objects in tree {tree_sha1}:")
for obj in sorted(all_tree_objects):
    print(obj)
```

**ASCII Diagram:**

```
Tree (tree_sha1)
│
├─ Blob (sha1_1)
├─ Blob (sha1_2)
└─ Tree (subtree_sha1)
    ├─ Blob (sha1_3)
    └─ Blob (sha1_4)
```

The function recursively descends into subtrees, collecting all blob and tree hashes.

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively finds and returns a set of all SHA-1 hashes of objects included in a commit, including the commit itself, its associated tree, and all parent commits.

**Parameters:**  
- `commit_sha1` (`str`): The SHA-1 hash (hex string) of the commit object to traverse.

**Returns:**  
- `set[str]`: A set of SHA-1 hex strings representing all objects reachable from the commit.

**Operation:**  
1. Initialize a set containing the current commit's SHA-1.  
2. Read and parse the commit object using `read_object(commit_sha1)`; confirm it is of type `'commit'`.  
3. Extract the tree SHA-1 from the commit data line starting with `'tree '`.  
4. Use `find_tree_objects(tree_sha1)` to recursively find all objects in the associated tree, adding them to the set.  
5. Extract all parent commit SHA-1s from lines starting with `'parent '`.  
6. Recursively call `find_commit_objects` for each parent commit and add their objects to the set.  
7. Return the aggregated set of objects.

**Example Usage:**

```python
commit_sha1 = "4b825dc642cb6eb9a060e54bf8d69288fbee4904"
all_commit_objects = find_commit_objects(commit_sha1)
print(f"Objects in commit {commit_sha1}:")
for obj in sorted(all_commit_objects):
    print(obj)
```

**ASCII Diagram:**

```
Commit (commit_sha1)
│
├─ Tree (tree_sha1)
│   ├─ Blob (blob_sha1_1)
│   └─ Tree (subtree_sha1)
│       └─ Blob (blob_sha1_2)
└─ Parent Commit (parent_sha1)
    ├─ Tree (parent_tree_sha1)
    └─ Parent Commit (...)
```

This diagram illustrates the recursive nature of commit object traversal, including parent commits and their trees.

---

## Related Functions (for context)

- `read_tree(sha1)`: Reads a tree object and returns a list of `(mode, path, sha1)` tuples.
- `read_object(sha1_prefix)`: Reads an object by SHA-1 prefix, returning its type and data.

---

## Summary

The `find_tree_objects` and `find_commit_objects` functions provide a comprehensive way to enumerate all Git objects reachable from a tree or commit. They are fundamental for operations that require knowledge of the full object graph, such as verifying repository integrity, preparing packs for network transfer, or garbage collection.

By recursively traversing trees and parent commits, these functions ensure no related object is missed, enabling robust and accurate repository state management.