# trees_commits.md

---

## Overview

This document covers the core functionalities related to managing Git tree and commit objects within the repository. It details how to read tree objects, write tree objects based on the current index, recursively find all objects referenced by trees and commits, and how to commit changes to the repository. These operations form the backbone of tracking project snapshots and history in Git, bridging the gap between the working directory, the index, and the commit history.

Positioned within the **Tree and Commit Management** section of the documentation, this file complements other modules that handle the index, object storage, and repository initialization by focusing specifically on the representation and manipulation of directory trees and commits.

---

## Function Documentation

### `read_tree(sha1=None, data=None)`

**Purpose:**  
Parses a Git tree object and returns a list of its entries. Each entry is a tuple containing the mode (file type and permissions), path (filename), and SHA-1 hash of the referenced object.

**Parameters:**  
- `sha1` (str, optional): The SHA-1 hash of the tree object to read.  
- `data` (bytes, optional): Raw data of the tree object (if already available).  

**Preconditions:**  
Either `sha1` or `data` must be provided. If `sha1` is given, the tree object is read and decompressed internally.

**Operation Steps:**  
1. If `sha1` is provided, use `read_object` to fetch the object data and assert it is a tree.  
2. Iterate over the raw tree data:  
   - Parse mode and path strings separated by a null byte (`\x00`).  
   - Extract 20 bytes after the null byte as the SHA-1 digest of the object.  
3. Collect all entries in a list, each as `(mode, path, sha1_hex)`.

**Example Usage:**

```python
# Read a tree object by its SHA-1
tree_sha1 = "4b825dc642cb6eb9a060e54bf8d69288fbee4904"
entries = read_tree(sha1=tree_sha1)
for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

---

### `write_tree()`

**Purpose:**  
Generates and writes a Git tree object from the current index entries (staged files), returning the SHA-1 hash of the new tree object.

**Parameters:**  
None.

**Preconditions:**  
- The index must be up to date and contain entries representing files to include in the tree.  
- Currently supports only a flat directory structure (no nested directories).

**Operation Steps:**  
1. Read the current index entries.  
2. For each entry, construct a tree entry byte sequence consisting of:  
   - File mode and path as ASCII encoded string (`"{mode:o} {path}"`),  
   - Null byte separator (`\x00`),  
   - The 20-byte SHA-1 binary digest of the object.  
3. Concatenate all tree entries and hash the combined data as a `tree` object via `hash_object`.  
4. Return the resulting SHA-1 hash.

**Example Usage:**

```python
# Write tree from index and get its SHA-1 hash
tree_hash = write_tree()
print(f"Tree object created with SHA-1: {tree_hash}")
```

---

### `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds all objects referenced by a tree object, including subtrees and blobs, and returns a set of their SHA-1 hashes.

**Parameters:**  
- `tree_sha1` (str): SHA-1 hash of the tree object to traverse.

**Operation Steps:**  
1. Initialize a set with the `tree_sha1` itself.  
2. Read the tree entries using `read_tree`.  
3. For each entry:  
   - If the mode indicates a directory (subtree), recursively collect its objects via `find_tree_objects`.  
   - Otherwise, add the blob SHA-1 to the set.  
4. Return the set containing all referenced object hashes.

**Example Usage:**

```python
# Find all objects in a tree and its subtrees
all_objects = find_tree_objects(tree_sha1="4b825dc642cb6eb9a060e54bf8d69288fbee4904")
print(f"Objects found in tree: {all_objects}")
```

**ASCII Diagram:**

```
Tree Object (SHA-1)
  |
  |-- Blob Object (file1)
  |
  |-- Tree Object (subdirectory)
         |
         |-- Blob Object (file2)
         |-- Blob Object (file3)
```

---

### `find_commit_objects(commit_sha1)`

**Purpose:**  
Recursively finds all objects reachable from a commit, including the commit itself, its tree, and all parent commits and their trees.

**Parameters:**  
- `commit_sha1` (str): SHA-1 hash of the commit object.

**Operation Steps:**  
1. Initialize a set with the commit SHA-1.  
2. Read and decode the commit object using `read_object`.  
3. Parse commit metadata to find:  
   - The tree hash.  
   - Zero or more parent commit hashes.  
4. Add all objects from the tree recursively via `find_tree_objects`.  
5. For each parent commit, recursively add its objects via `find_commit_objects`.  
6. Return the set of all collected SHA-1 hashes.

**Example Usage:**

```python
# Get all objects referenced by a commit and its ancestors
commit_hash = "c1a414b2ff5b2a9e4a8b5d2e3f7e1d2c4f5a6b7c"
objects = find_commit_objects(commit_hash)
print(f"All objects reachable from commit: {objects}")
```

---

### `commit(message, author=None)`

**Purpose:**  
Creates a new commit object capturing the current state of the index, updates the local `master` branch reference, and returns the commit's SHA-1 hash.

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author information in the format `"Name <email>"`. If omitted, environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL` are used.

**Operation Steps:**  
1. Write the current index to a tree object, getting its SHA-1.  
2. Retrieve the current commit hash on the `master` branch (parent commit).  
3. Construct author and committer metadata with timestamps and timezone offsets.  
4. Assemble commit object content including:  
   - Tree hash,  
   - Parent commit hash (if any),  
   - Author and committer info,  
   - Commit message.  
5. Hash and write the commit object to the object store.  
6. Update `refs/heads/master` to point to the new commit hash.  
7. Print confirmation of the commit and return the commit SHA-1.

**Example Usage:**

```python
# Commit staged changes with a custom message
new_commit_sha1 = commit("Fix bug in authentication flow")
print(f"New commit created: {new_commit_sha1}")
```

---

## Summary ASCII Diagram: Trees and Commits Relationship

```
+------------------------+
|       Commit Object    |  <-- points to root tree + parent commits
|  --------------------  |
|  tree <tree_sha1>       |
|  parent <parent_sha1>   |
|  author ...             |
|  committer ...          |
+-----------+------------+
            |
            v
+------------------------+
|        Tree Object      |  <-- contains blobs and subtrees
|  mode path sha1         |
|  mode path sha1         |
+-----------+------------+
            |
            +----> Blob Object (file contents)
            |
            +----> Tree Object (subdirectory)
                     |
                     +----> Blob Object
```

---

This documentation provides a detailed explanation of the key functions for managing Git tree and commit objects, essential for understanding how snapshots and commit history are constructed and maintained in a Git repository.