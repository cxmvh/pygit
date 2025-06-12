# object_traversal.md

## Overview

This document describes the core functions responsible for recursively discovering Git objects within commits and trees, as well as encoding these objects for inclusion in pack files. These operations are crucial in managing the Git object graph, enabling efficient traversal of commits and their associated trees and blobs, and preparing objects for transmission or storage in compressed packfiles. This file fits within the broader **Object Handling and Git Objects** section of the repository documentation, supporting commands such as `cat-file` and operations related to packfile creation and transfer.

---

## Function Documentation

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively find all Git objects reachable from a given commit. This includes the commit object itself, its associated tree, all objects reachable from that tree (trees and blobs), and all parent commits and their reachable objects.

**Parameters:**  
- `commit_sha1` (str): The SHA-1 hash of the commit object to start traversal from.

**Returns:**  
- `Set[str]`: A set of SHA-1 hex strings representing all objects reachable from the commit.

**Operation Steps:**
1. Initialize a set `objects` containing the commit SHA-1.
2. Read the object corresponding to `commit_sha1` and assert it is a commit.
3. Parse the commit data to find the SHA-1 of its tree.
4. Recursively call `find_tree_objects` on the tree SHA-1 to get all objects in the tree, add them to `objects`.
5. Extract parent commit SHA-1s from the commit data.
6. For each parent commit, recursively call `find_commit_objects` and add all returned objects to `objects`.
7. Return the complete set of objects.

**Example Usage:**
```python
commit_hash = "a1b2c3d4e5f6g7h8i9j0k1234567890abcdef1234"
all_objects = find_commit_objects(commit_hash)
print(f"All objects reachable from commit {commit_hash}:")
for obj in all_objects:
    print(obj)
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively find all Git objects contained in a tree object. This includes the tree itself, all subtrees, and blob objects (file contents) referenced by the tree.

**Parameters:**  
- `tree_sha1` (str): The SHA-1 hash of the tree object to traverse.

**Returns:**  
- `Set[str]`: A set of SHA-1 hex strings of all objects reachable from the tree.

**Operation Steps:**
1. Initialize a set `objects` with the SHA-1 of the given tree.
2. Read the tree object entries (mode, path, SHA-1).
3. For each entry:
   - If the entry is a directory (subtree), recursively call `find_tree_objects` and add all returned objects.
   - Otherwise, add the blob SHA-1 to `objects`.
4. Return the set of all found objects.

**Example Usage:**
```python
tree_hash = "1a2b3c4d5e6f7g8h9i0jklmnopqrstuvwx"
tree_objects = find_tree_objects(tree_hash)
print(f"Objects in tree {tree_hash}:")
for obj in tree_objects:
    print(obj)
```

---

### `encode_pack_object(obj)`

**Purpose:**  
Encode a single Git object into the packfile format. This involves creating a variable-length header based on the object type and size, followed by the zlib-compressed object data.

**Parameters:**  
- `obj` (str): The SHA-1 hash of the object to encode.

**Returns:**  
- `bytes`: The encoded packfile object data including the header and compressed object data.

**Operation Steps:**
1. Read the object type and raw data for the given SHA-1.
2. Map the object type to its numerical code (`ObjectType` enum).
3. Calculate the size of the data.
4. Create the first byte of the header encoding the object type and lower 4 bits of size.
5. Continue encoding the size in 7-bit chunks, setting the continuation bit if more bytes follow.
6. Append the final size byte without the continuation bit.
7. Compress the object data using zlib.
8. Return the concatenation of the header bytes and compressed data.

**Example Usage:**
```python
pack_data = encode_pack_object("abc123def4567890abc123def4567890abc123de")
with open("object.pack", "wb") as f:
    f.write(pack_data)
```

---

### ASCII Diagram: Recursive Object Traversal

The following diagram shows the recursive traversal of commit and tree objects to collect all reachable Git objects:

```
Commit (commit_sha1)
  |
  +--> Tree (tree_sha1)
  |       |
  |       +--> Blob (file1)
  |       |
  |       +--> Tree (subtree_sha1)
  |               |
  |               +--> Blob (file2)
  |
  +--> Parent Commit (parent_sha1)
          |
          +--> Tree (parent_tree_sha1)
          |       |
          |       +--> Blob (file3)
          |
          +--> Parent Commit (grandparent_sha1)
                  ...
```

---

## Summary

The functions in `object_traversal.md` enable deep inspection and packaging of Git objects by:

- Recursively finding all objects reachable from a commit or tree.
- Encoding objects for efficient storage and transmission in packfiles.

These operations underpin Git’s ability to track object graphs and support commands that manipulate and transport repository data effectively.