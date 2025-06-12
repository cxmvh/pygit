# tree.md

# Writing and Reading Tree Objects

---

## Overview

The `tree.md` file documents the functionality related to writing and reading Git tree objects within the `pygit` project. Tree objects in Git represent directory contents and link to blobs (file contents) or other trees (subdirectories). This file is part of the **Core Git Commands** section, specifically under **Index and Tree Management**, and plays a critical role in capturing the state of the working directory within commits.

This documentation explains how the system serializes the index entries into tree objects, how it reads tree objects back, and how these operations integrate with the commit process. Understanding trees is fundamental for grasping how Git stores snapshots of the project at various points in time.

---

## Important Functions

### 1. `write_tree()`

#### Purpose

Creates and writes a Git tree object from the current index entries, then returns the SHA-1 hash of the created tree object.

- Converts each file entry in the index to a tree entry.
- Concatenates these entries into the tree object data.
- Hashes and writes the tree object to the Git object store.

#### Parameters

- None

#### Preconditions

- The Git index must be populated with file entries.
- Currently supports only a single, top-level directory (no nested subdirectories).

#### How It Works

1. Reads all entries from the index using `read_index()`.
2. For each entry:
   - Asserts that the file path does not contain a slash (`/`), ensuring top-level only.
   - Formats the mode and path as a string, then encodes it.
   - Concatenates the mode/path encoding with a null byte (`\x00`), then appends the SHA-1 binary hash of the blob.
3. Joins all tree entries into a single byte string.
4. Passes the byte string to `hash_object()` with object type `'tree'` to:
   - Create the tree object.
   - Write it to the Git object store.
   - Return its SHA-1 hash.

#### Returns

- SHA-1 hash string of the written tree object.

#### Example Usage

```python
tree_sha1 = write_tree()
print(f"Tree object created with SHA-1: {tree_sha1}")
```

---

### 2. `read_tree(sha1)`

*(Note: This function is referenced in the code but not explicitly provided in the snippet. Below is an inferred explanation based on typical Git tree reading behavior.)*

#### Purpose

Reads a Git tree object by its SHA-1 hash and returns its entries, enabling inspection or traversal.

#### Parameters

- `sha1` (str): SHA-1 hash of the tree object to read.

#### Returns

- Iterable of tuples `(mode, path, sha1_bytes)` representing each entry in the tree:
  - `mode` (int): File mode bits.
  - `path` (str): Filename.
  - `sha1_bytes` (bytes): SHA-1 hash of the linked object (blob or subtree).

#### How It Works

1. Reads the raw tree object data using `read_object(sha1)`.
2. Parses the raw byte stream into entries by:
   - Extracting mode (as ASCII octal), path (filename), and SHA-1 hash (20 bytes).
   - Iterating until the entire tree data is consumed.

#### Example Usage

```python
for mode, path, sha1 in read_tree(tree_sha1):
    print(f"Mode: {oct(mode)}, Path: {path}, Object SHA-1: {sha1.hex()}")
```

---

### 3. `find_tree_objects(tree_sha1)`

#### Purpose

Recursively finds all objects (blobs and trees) contained within a tree object, including the tree itself.

This function helps to enumerate every Git object referenced by a given tree, which is essential for operations like pushing commits or creating packfiles.

#### Parameters

- `tree_sha1` (str): SHA-1 hash of the tree object to analyze.

#### How It Works

1. Initializes a set with the given tree SHA-1.
2. Reads entries of the tree using `read_tree(tree_sha1)`.
3. For each entry:
   - Checks if the mode represents a directory (`stat.S_ISDIR(mode)`).
   - If a directory, recursively calls `find_tree_objects()` on the subtree SHA-1 and updates the set.
   - Otherwise, adds the blob SHA-1 to the set.
4. Returns the complete set of SHA-1 hashes representing all objects reachable from the tree.

#### Returns

- A `set` of SHA-1 hashes (strings or bytes) including the original tree and all its contents recursively.

#### Example Usage

```python
all_objects = find_tree_objects(tree_sha1)
print(f"Total objects in tree: {len(all_objects)}")
for obj_sha1 in all_objects:
    print(obj_sha1)
```

---

## Integration Diagram: Tree Writing in Commit Flow

```
+-----------------+       +----------------+       +-------------------+
| Read Index      |  -->  | write_tree()   |  -->  | hash_object('tree')|
| (read_index())  |       | (creates tree) |       | (writes tree obj) |
+-----------------+       +----------------+       +-------------------+
        |                          |                          |
        |                          |                          |
        |                          |                          |
        |                          v                          v
        |                 Tree SHA-1 hash returned     Tree object stored
        |                          |
        +--------------------------+
                                   |
                                   v
                          +----------------+
                          | commit()       |
                          | Uses tree SHA-1|
                          +----------------+
```

---

## Summary

- The `write_tree()` function converts the current index state into a serialized tree object, which is a core step in creating a Git commit snapshot.
- `find_tree_objects()` provides recursive traversal of tree objects to enumerate referenced blobs and subtrees.
- Together, these functions enable the storage and retrieval of directory structures within Git's object database.

This file is essential for developers working on Git internals related to tree management, commit creation, or object traversal in `pygit`.