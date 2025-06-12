# commits.md

# Commit Object Handling and Related Functions

---

## Overview

The `commits.md` file documents the handling of Git commit objects within the pygit project. It resides under the **Trees and Commits** section of the overall documentation tree, which focuses on operations related to tree and commit objects, including their recursive processing. This file plays a crucial role in explaining how commit objects are read, created, traversed, and linked to other Git objects like trees and parent commits. Understanding commit object handling is fundamental to grasping how pygit tracks repository history, manages snapshots, and supports core Git functionalities such as branching, committing, and pushing.

---

## Important Functions

### 1. `cat_file(mode, sha1_prefix)`

#### Purpose

Outputs the raw or processed content of a Git object identified by its SHA-1 prefix. It supports multiple display modes such as showing the raw content of commits, trees, and blobs, printing the size or type of an object, or providing a pretty-printed format.

#### Parameters

- `mode` (str): The display mode. Possible values:
  - `'commit'`, `'tree'`, `'blob'`: Print raw data of the object.
  - `'size'`: Print the size of the object data.
  - `'type'`: Print the object type.
  - `'pretty'`: Print a user-friendly representation of the object.
- `sha1_prefix` (str): The SHA-1 hash prefix identifying the object.

#### Operation

1. Reads the object type and data using `read_object(sha1_prefix)`.
2. Depending on `mode`:
   - For `'commit'`, `'tree'`, `'blob'`, verifies the object type matches the mode and writes raw data to stdout.
   - For `'size'`, prints the length of data.
   - For `'type'`, prints the object's type.
   - For `'pretty'`:
     - For commits and blobs, prints raw data.
     - For trees, iterates over tree entries and prints each entry with details.

#### Usage Example

```python
# Display raw commit content
cat_file('commit', 'a1b2c3')

# Show object size
cat_file('size', 'a1b2c3')

# Pretty-print a tree object
cat_file('pretty', 'd4e5f6')
```

---

### 2. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the object store by SHA-1 prefix and returns its type and raw data.

#### Parameters

- `sha1_prefix` (str): The SHA-1 prefix identifying the object.

#### Operation

1. Finds the full object path using `find_object(sha1_prefix)`.
2. Reads and decompresses the object data.
3. Parses the header to extract object type and size.
4. Returns a tuple `(object_type, data_bytes)`.

#### Usage Example

```python
obj_type, data = read_object('a1b2c3')
print(f"Object type: {obj_type}, Data size: {len(data)} bytes")
```

---

### 3. `read_tree(sha1=None, data=None)`

#### Purpose

Parses a tree object, returning a list of entries (mode, path, sha1).

#### Parameters

- `sha1` (str, optional): SHA-1 hash of the tree object.
- `data` (bytes, optional): Raw tree data bytes. At least one of `sha1` or `data` must be provided.

#### Operation

1. If `sha1` is given, read the tree object data.
2. Iteratively parse entries until no more are found:
   - Each entry contains a mode, path, and SHA-1 hash.
3. Returns a list of `(mode, path, sha1)` tuples.

#### Usage Example

```python
entries = read_tree(sha1='d4e5f6')
for mode, path, sha1 in entries:
    print(f"{mode:o} {path} {sha1}")
```

---

### 4. `find_commit_objects(commit_sha1)`

#### Purpose

Recursively finds all Git objects (commits, trees, blobs) reachable from a given commit SHA-1, including its parent commits and trees.

#### Parameters

- `commit_sha1` (str): SHA-1 hash of the starting commit object.

#### Operation

1. Initialize a set with the commit SHA-1.
2. Read the commit object and extract the tree SHA-1.
3. Recursively find all objects in the tree using `find_tree_objects(tree)`.
4. For each parent commit, recursively find their objects.
5. Return a set of all reachable object SHA-1s.

#### Usage Example

```python
objects = find_commit_objects('a1b2c3')
print(f"Total reachable objects: {len(objects)}")
```

---

### 5. `find_tree_objects(tree_sha1)`

#### Purpose

Recursively finds all objects contained within a given tree SHA-1, including nested trees and blob objects.

#### Parameters

- `tree_sha1` (str): SHA-1 hash of the tree object.

#### Operation

1. Initialize a set with the tree SHA-1.
2. Read the tree entries.
3. For each entry:
   - If a directory (tree), recursively find its objects.
   - Else, add the blob SHA-1.
4. Return the complete set of object SHA-1s.

#### Usage Example

```python
tree_objects = find_tree_objects('d4e5f6')
print(f"Objects in tree: {tree_objects}")
```

---

### 6. `commit(message, author=None)`

#### Purpose

Creates a new commit object from the current index state, writes it to the object store, and updates the local master branch reference.

#### Parameters

- `message` (str): Commit message.
- `author` (str, optional): Author string in the form `"Name <email>"`. If not provided, retrieved from environment variables.

#### Operation

1. Writes the current index to a tree object (`write_tree()`).
2. Retrieves the current master branch commit hash (`get_local_master_hash()`).
3. Constructs commit metadata: tree, parent(s), author, committer, timestamp.
4. Composes commit object content.
5. Hashes and writes the commit object (`hash_object()`).
6. Updates the `.git/refs/heads/master` reference.
7. Prints confirmation and returns the commit SHA-1.

#### Usage Example

```python
sha1 = commit("Add new feature", author="Alice <alice@example.com>")
print(f"Created commit {sha1}")
```

---

### 7. `write_tree()`

#### Purpose

Creates a tree object from the current index entries, representing the snapshot of the working directory at commit time.

#### Parameters

None.

#### Operation

1. Reads current index entries.
2. For each entry, constructs a tree entry with mode, path, and SHA-1.
3. Concatenates all entries and hashes the resulting data as a tree object.
4. Writes the tree object to the object store.
5. Returns the SHA-1 hash of the new tree.

#### Usage Example

```python
tree_sha1 = write_tree()
print(f"Tree object created with SHA-1: {tree_sha1}")
```

---

### ASCII Diagram: Commit Object Structure and Relationships

```
+----------------+
|    Commit      |  <---- points to ----
|  (commit SHA-1)|                        \
+----------------+                         \
| tree: <tree SHA-1> ------------------> +----------------+ 
| parent: <parent SHA-1> (optional)       |     Tree       |
| author: ...                             |  (tree SHA-1)  |
| committer: ...                         +----------------+
| message: ...                           /        |    \
+----------------+                     /         |     \
                                   contains   contains  contains
                                    blobs      blobs     trees
                                     |          |         |
                                +----------+ +----------+ +----------+
                                |  Blob    | |  Blob    | |  Tree    |
                                | (file)   | | (file)   | | (subtree)|
                                +----------+ +----------+ +----------+
```

This diagram illustrates how a commit references a tree object capturing the directory snapshot, which in turn references blob objects (files) and possibly other tree objects (subdirectories).

---

## Summary

This documentation file covers the key functions involved in handling commit objects within pygit. It explains how commit objects are read and created, how they relate recursively to tree and blob objects, and how the commit history is traversed. The functions documented here underpin essential Git operations such as creating commits, inspecting commit contents, and traversing commit graphs, thus forming a cornerstone of pygit's Git object management capabilities.