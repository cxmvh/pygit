# Tree and Commit Objects Management

This document describes functions for reading and finding git tree and commit objects within the pygit repository. It provides detailed explanations and usage examples for the key functions `read_tree`, `find_tree_objects`, and `find_commit_objects`. These functions are essential for navigating the git object database, enabling recursive exploration of trees and commits and their associated objects.

This file fits into the broader documentation section **Tree and Commit Object Management**, which focuses on handling tree and commit objects, including reading trees and finding related objects. These functionalities complement other parts of pygit such as object hashing, indexing, committing, and pushing changes.

---

## Function: read_tree

### Purpose

Parse a git tree object by SHA-1 hash or raw data and return its entries. Each entry corresponds to a file or directory in that tree and includes its mode, path, and SHA-1 hash.

### Parameters

- `sha1` (str, optional): SHA-1 hex string of the tree object to read.
- `data` (bytes, optional): Raw tree object data bytes. If provided, `sha1` is ignored.

At least one of `sha1` or `data` must be provided.

### Returns

- List of tuples `(mode, path, sha1)` where:
  - `mode` (int): File mode (e.g., 100644 for file, 40000 for directory).
  - `path` (str): Filename or directory name.
  - `sha1` (str): SHA-1 hex string of the object (blob or tree).

### Operation

1. If `sha1` is specified, use `read_object(sha1)` to get the tree's raw data.
2. Parse the raw tree data, which is a concatenation of entries:
   - Each entry: `<mode> <filename>\0<20-byte SHA-1>`
3. Extract `mode`, `path`, and SHA-1 digest for each entry.
4. Return all parsed entries as a list.

### Example Usage

```python
tree_sha1 = '4a202b346bb0fb0db7eff3cffeb3c70babbd2045'
entries = read_tree(sha1=tree_sha1)
for mode, path, sha1 in entries:
    print(f'{oct(mode)} {path} {sha1}')
```

### ASCII Diagram of Tree Object Structure

```
+---------------------------+
| Tree Object Raw Data      |
+---------------------------+
| mode path\0 SHA-1 (20b)   |  <-- repeated entries
| mode path\0 SHA-1 (20b)   |
| ...                       |
+---------------------------+
```

---

## Function: find_tree_objects

### Purpose

Recursively find all git objects (trees and blobs) referenced by a tree object, including the tree itself and all nested subtrees.

### Parameters

- `tree_sha1` (str): SHA-1 hex string of the tree object.

### Returns

- Set of SHA-1 hex strings of all objects referenced by the tree.

### Operation

1. Initialize a set `objects` with the `tree_sha1`.
2. Read the tree entries using `read_tree(tree_sha1)`.
3. For each entry:
   - If the entry is a directory (mode indicates a tree), recursively call `find_tree_objects` on its SHA-1 and add results to `objects`.
   - Otherwise, add the blob SHA-1 to `objects`.
4. Return the complete set.

### Example Usage

```python
tree_sha1 = '4a202b346bb0fb0db7eff3cffeb3c70babbd2045'
all_objects = find_tree_objects(tree_sha1)
print(f'Objects in tree {tree_sha1}:')
for obj in sorted(all_objects):
    print(obj)
```

### ASCII Diagram of Recursive Tree Exploration

```
tree_sha1
   |
   +-- blob_sha1 (file)
   |
   +-- subtree_sha1 (directory)
           |
           +-- blob_sha1 (file)
           +-- subtree_sha1 (directory)
                   |
                   +-- blob_sha1 (file)
```

---

## Function: find_commit_objects

### Purpose

Recursively find all git objects referenced by a commit object, including the commit itself, its tree, and all parent commits and their trees.

### Parameters

- `commit_sha1` (str): SHA-1 hex string of the commit object.

### Returns

- Set of SHA-1 hex strings of all objects referenced by the commit.

### Operation

1. Initialize a set `objects` with the `commit_sha1`.
2. Read the commit object data using `read_object(commit_sha1)`.
3. Parse commit lines to find:
   - The tree SHA-1.
   - Zero or more parent commit SHA-1s.
4. Add all objects from the tree (recursively) by calling `find_tree_objects(tree)`.
5. For each parent commit, recursively call `find_commit_objects(parent)` and add results.
6. Return the complete set.

### Example Usage

```python
commit_sha1 = '5f1d7f77e1a77b5a9d1c3d8f0f2b6d8c1a2b3456'
commit_objects = find_commit_objects(commit_sha1)
print(f'Objects reachable from commit {commit_sha1}:')
for obj in sorted(commit_objects):
    print(obj)
```

### ASCII Diagram of Commit Object References

```
commit_sha1
   |
   +-- tree_sha1
   |       |
   |       +-- (all tree objects recursively)
   |
   +-- parent_sha1 (commit)
           |
           +-- tree_sha1
           |       |
           |       +-- (all tree objects recursively)
           |
           +-- (possibly more parents recursively)
```

---

# Summary

The functions in this module provide essential capabilities to read and explore git tree and commit objects. With `read_tree`, you can parse the contents of a tree object. The recursive functions `find_tree_objects` and `find_commit_objects` allow collecting all objects reachable from trees and commits respectively, facilitating operations like packfile creation, object verification, or repository traversal.

These functions integrate with other pygit components such as `read_object` (for reading raw git objects) and `hash_object` (for creating objects), supporting a complete git object handling workflow.