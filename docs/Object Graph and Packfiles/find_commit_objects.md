# find_commit_objects.md

## Overview

This document details the functionality for **finding all objects reachable from a given commit** in a Git repository. It is part of the **Object Graph and Packfiles** section, which focuses on managing the commit graph, object sets, and packfile creation. Understanding how to traverse the commit and its related trees and parent commits is essential for operations such as determining the complete object set involved in a commit, creating packfiles for pushing to remotes, and verifying repository integrity.

The key function documented here is `find_commit_objects(commit_sha1)`, which recursively resolves all objects reachable from a commit, including its tree, all nested trees and blobs, parent commits, and the commit itself.

---

## Function Documentation

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Return a set of SHA-1 hashes for all Git objects reachable from the specified commit. This includes:

- The commit object itself.
- The tree object referenced by the commit.
- All blobs and trees recursively reachable from the tree.
- All parent commits and their reachable objects recursively.

This function is fundamental for operations that require knowledge of the full object graph starting from a commit, such as constructing packfiles for pushing or fetching.

**Parameters:**

- `commit_sha1` (str): The SHA-1 hash string of the commit object to start from.

**Returns:**

- `set[str]`: A set of SHA-1 hex strings representing all objects reachable from the commit.

**Preconditions:**

- The given `commit_sha1` must correspond to a valid commit object in the local Git object store.
- The repository must be intact with accessible objects.

**Operation Steps:**

1. Initialize an object set containing the initial commit SHA-1.
2. Read the commit object data using the hash:
   - Confirm the object is of type `commit`.
3. Parse the commit data to extract:
   - The SHA-1 of the root tree object.
   - Zero or more parent commit SHA-1 hashes.
4. Recursively call `find_tree_objects(tree_sha1)` to find all objects within the tree (and nested trees/blobs).
5. For each parent commit found, recursively call `find_commit_objects(parent_sha1)` to include all objects reachable from the parents.
6. Combine all these objects into a single set and return it.

---

**Code Example:**

```python
def find_commit_objects(commit_sha1):
    """Return set of SHA-1 hashes of all objects in this commit (recursively),
    its tree, its parents, and the hash of the commit itself.
    """
    objects = {commit_sha1}
    obj_type, commit = read_object(commit_sha1)
    assert obj_type == 'commit'
    lines = commit.decode().splitlines()
    tree = next(l[5:45] for l in lines if l.startswith('tree '))
    objects.update(find_tree_objects(tree))
    parents = (l[7:47] for l in lines if l.startswith('parent '))
    for parent in parents:
        objects.update(find_commit_objects(parent))
    return objects
```

---

### Supporting Function: `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively find all objects reachable from a tree object. This includes the tree itself, all blobs, and nested trees.

**Summary:**  
- Adds the tree SHA-1 to the object set.
- Reads the tree contents to get entries (mode, path, sha1).
- If an entry is a tree (directory), recursively calls itself.
- Adds blob SHA-1s directly to the set.

---

### ASCII Diagram: Object Graph Traversal from Commit

```
Commit object (commit_sha1)
│
├── Tree object (tree_sha1)
│   ├── Blob object(s)
│   ├── Tree object(s) (nested)
│        ├── Blob object(s)
│        └── Tree object(s) ...
│
└── Parent commit(s)
    ├── Tree object(s)
    │    └── Blob/tree objects recursively
    └── Parent commit(s) recursively
```

This diagram illustrates how starting at a commit, the traversal proceeds down through the tree objects and up through parent commits, recursively gathering all associated Git objects.

---

### Usage Example

Suppose you want to find all objects reachable from a commit with SHA-1 hash `a1b2c3d4e5f6g7h8i9j0klmnopqrstuvwx1234`:

```python
commit_hash = 'a1b2c3d4e5f6g7h8i9j0klmnopqrstuvwx1234'
all_objects = find_commit_objects(commit_hash)

print(f"Total objects reachable from commit {commit_hash}: {len(all_objects)}")
for obj in sorted(all_objects):
    print(obj)
```

This will print the full set of object hashes (commit, trees, blobs, parent commits) reachable from the given commit.

---

## Summary

- `find_commit_objects` is a recursive function to collect all Git objects reachable from a commit.
- It relies on reading commit and tree objects and recursively traversing parents and nested trees.
- This function is essential for packfile creation, syncing objects between repositories, and understanding the commit's full object graph.

---

# End of `find_commit_objects.md` documentation file content.