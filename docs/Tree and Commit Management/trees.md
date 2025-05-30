# trees.md

# Reading and Writing Tree Objects Recursively

---

## Overview

The `trees.md` document details the functionality for recursively reading and writing Git tree objects within the `pygit` project. Tree objects in Git represent directory structures, mapping file names to blob objects or subtrees. This file plays a critical role in the **Tree and Commit Management** section of the documentation, complementing other files such as `trees_and_commits.md` and `commit.md`.

In the broader project context, these recursive tree operations enable efficient tracking and manipulation of directory contents, which is foundational for commit creation, diff operations, and repository state management. The functions described here form the backbone for serializing and deserializing directory hierarchies into Git's object database.

---

## Important Functions

### 1. `read_tree(sha1=None, data=None)`

**Purpose:**  
Reads a Git tree object either from a SHA-1 hash or raw data bytes and parses it into a list of entries. Each entry corresponds to a file or directory, represented by its mode, path, and SHA-1 hash.

**Parameters:**
- `sha1` (str, optional): The SHA-1 hex string of the tree object to read.
- `data` (bytes, optional): Raw bytes of the tree object data. Must be specified if `sha1` is not given.

**Returns:**  
A list of tuples `(mode, path, sha1)` where:  
- `mode` (int): File mode (e.g., `100644` for normal files, `40000` for directories).  
- `path` (str): File or directory path relative to the tree root.  
- `sha1` (str): SHA-1 hash of the object (blob or subtree).

**Operation:**  
1. If `sha1` is provided, the object is read and decompressed from Git's object database.
2. The data is scanned sequentially:
   - Each entry starts with a mode and path string separated by a space, ending with a null byte.
   - Following the null byte is a 20-byte SHA-1 digest.
3. The function loops until no more entries are found.

**Usage Example:**

```python
# Read tree by SHA-1
entries = read_tree(sha1='a1b2c3d4e5f6g7h8i9j0k1234567890abcdef1234')

for mode, path, sha1 in entries:
    print(f"Mode: {oct(mode)}, Path: {path}, SHA-1: {sha1}")
```

**ASCII Diagram:**

```
+----------------------------+
| Tree Object Data Structure |
+----------------------------+

[mode path\x00 sha1][mode path\x00 sha1][...]
|    |        |    |     |
|    |        |    |     +--> 20-byte SHA-1 digest
|    |        |    +--------> null byte (0x00) separator
|    |        +-------------> path string (filename or directory)
|    +---------------------> mode string (octal number)
```

---

### 2. `write_tree()`

**Purpose:**  
Creates a tree object from the current Git index entries and writes it to the object store. Returns the SHA-1 hash of the written tree object.

**Parameters:**  
None

**Returns:**  
SHA-1 hash (hex string) of the written tree object.

**Operation:**  
1. Reads all index entries via `read_index()`.
2. Currently supports only a *single* top-level directory (no nested paths).
3. For each index entry:
   - Formats a string of the form `"{mode:o} {path}"`.
   - Concatenates it with a null byte and the SHA-1 binary digest of the blob.
4. Joins all such entries and hashes them as a `tree` object by calling `hash_object()`.
5. The resulting SHA-1 is returned.

**Usage Example:**

```python
# Write tree object from current index entries
tree_sha1 = write_tree()
print(f"Created tree object with SHA-1: {tree_sha1}")
```

---

### 3. `find_tree_objects(tree_sha1)`

**Purpose:**  
Recursively finds all object SHA-1 hashes referenced by a tree, including blobs and subtrees, returning a set of all referenced objects.

**Parameters:**  
- `tree_sha1` (str): SHA-1 hash of the tree object.

**Returns:**  
A `set` of SHA-1 strings representing the tree itself and all objects nested under it.

**Operation:**  
1. Adds the initial tree SHA-1 to a set.
2. Reads the tree entries using `read_tree()`.
3. For each entry:
   - If the mode indicates a directory (`stat.S_ISDIR(mode)`), recursively calls itself on the subtree SHA-1.
   - Otherwise, adds the blob SHA-1 to the set.
4. Returns the complete set.

**Usage Example:**

```python
# Recursively list all objects under a tree
objects = find_tree_objects(tree_sha1='a1b2c3d4e5f6g7h8i9j0k1234567890abcdef1234')
print("Objects in tree and subtrees:")
for obj in sorted(objects):
    print(obj)
```

**ASCII Diagram:**

```
Tree SHA-1
   |
   +-- blob SHA-1 (file)
   +-- subtree SHA-1
         |
         +-- blob SHA-1 (file)
         +-- blob SHA-1 (file)
```

---

### 4. `commit(message, author=None)`

**Purpose:**  
Creates a commit object from the current index state, writing all referenced tree and blob objects recursively, and updates the `master` branch reference.

**Parameters:**
- `message` (str): Commit message.
- `author` (str, optional): Author name and email as "Name <email>".  
  Defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Returns:**  
SHA-1 hash (hex string) of the created commit object.

**Operation:**  
1. Calls `write_tree()` to write the current tree object.
2. Retrieves the current `master` branch commit hash (if any) as parent.
3. Constructs commit metadata with author, committer, timestamps, and parents.
4. Creates the commit object and hashes it with `hash_object()`.
5. Updates the `refs/heads/master` file with the new commit SHA-1.
6. Prints the commit hash.

**Usage Example:**

```python
commit_hash = commit("Initial commit")
print(f"Committed with hash: {commit_hash}")
```

---

## Additional Context and Usage

The recursive reading and writing of tree objects is essential when performing operations like:

- Creating commits that reflect directory structures.
- Finding all objects reachable from a commit (used in `find_commit_objects` and during push).
- Generating diffs by comparing tree contents.
- Writing packfiles containing all necessary objects.

---

## Summary Diagram: Tree Object Recursive Handling

```
+---------------------------+
|         Commit            |
+---------------------------+
            |
            v
+---------------------------+
|         Tree SHA-1        | <--- write_tree() serializes index entries
+---------------------------+
            |
  +---------+----------+
  |                    |
  v                    v
Blob SHA-1           Subtree SHA-1
(file)                (directory)
                       |
                       +-- Blob SHA-1
                       +-- Subtree SHA-1 (further recursion)
```

This structure enables Git to efficiently represent nested directories and files.

---

# End of trees.md documentation content.