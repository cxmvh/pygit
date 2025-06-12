# commit.md

## Overview

This document provides a detailed explanation of the commit creation process in the `pygit` repository, focusing on the critical functions involved in committing the current index state to the master branch. It explains how the tree object is written from the index, how git objects are hashed and stored, and how commit objects are constructed and recorded. This file fits within the **Repository Initialization and Core Concepts** section of the documentation tree, linking core git operations with index management and object handling. Understanding these commit-related functions is essential for grasping how `pygit` manages repository state changes and persists them as commits.

---

## Function Documentation

### `commit(message, author=None)`

**Purpose:**

Commit the current state of the index to the `master` branch with a commit message and optional author information. Returns the SHA-1 hash of the created commit object.

**Parameters:**

- `message` (str): The commit message describing the changes.
- `author` (str, optional): The author string in the format `"Name <email>"`. If not provided, it uses environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation:**

1. Calls `write_tree()` to write the current index as a tree object and obtains its SHA-1.
2. Retrieves the current commit hash of the local `master` branch using `get_local_master_hash()` (to set as the parent commit).
3. Constructs the author and committer strings with timestamps and timezone offsets.
4. Builds the commit object data containing:
   - Tree hash
   - Parent commit hash (if any)
   - Author and committer information
   - Commit message
5. Uses `hash_object()` to hash and store the commit object.
6. Updates the `refs/heads/master` reference to point to the new commit.
7. Prints a confirmation message and returns the commit SHA-1.

**Example Usage:**

```python
commit_sha1 = commit("Initial commit")
print(f"New commit created: {commit_sha1}")
```

**ASCII Diagram:**

```
Current Index State
        |
   [write_tree()]
        |
   Tree Object SHA-1
        |
   [commit()]
        |
  +-----------------------------+
  | commit object data          |
  |  - tree <tree_sha1>         |
  |  - parent <parent_sha1>     |
  |  - author <author>          |
  |  - committer <committer>    |
  |  - message                 |
  +-----------------------------+
        |
   hash_object() -> commit SHA-1
        |
 Update master ref -> new commit
```

---

### `write_tree()`

**Purpose:**

Creates a tree object from the current index entries, representing the snapshot of the working directory’s tracked files at the time of commit.

**Parameters:** None

**Operation:**

1. Reads the list of index entries via `read_index()`.
2. For each entry (assumed to be top-level files only), constructs a tree entry:
   - Format: `<mode> <filename>\0<sha1>`
3. Joins all entries and hashes them as a `tree` object using `hash_object()`.
4. Returns the SHA-1 of the created tree object.

**Example Usage:**

```python
tree_sha1 = write_tree()
print(f"Tree object created: {tree_sha1}")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**

Compute the SHA-1 hash of the given data as a git object of the specified type and optionally write the compressed object to the git object database.

**Parameters:**

- `data` (bytes): The raw data to hash.
- `obj_type` (str): Git object type (`'blob'`, `'tree'`, `'commit'`, etc.).
- `write` (bool): Whether to write the object to storage (default `True`).

**Operation:**

1. Constructs the object header as `<obj_type> <len(data)>`.
2. Concatenates header, a null byte, and data.
3. Computes SHA-1 hash of the full data.
4. If `write` is `True`, compresses and stores the object in `.git/objects/`.
5. Returns the SHA-1 hash as a hex string.

**Example Usage:**

```python
blob_sha1 = hash_object(b'Hello, world!', 'blob')
print(f"Blob object SHA-1: {blob_sha1}")
```

---

### `get_local_master_hash()`

**Purpose:**

Retrieve the current commit hash of the local `master` branch.

**Parameters:** None

**Operation:**

1. Reads the file `.git/refs/heads/master`.
2. Returns the SHA-1 commit hash string.
3. Returns `None` if no commit exists yet (file not found).

**Example Usage:**

```python
current_master = get_local_master_hash()
print(f"Current master commit: {current_master}")
```

---

### `write_file(path, data)`

**Purpose:**

Write the given bytes data to the specified file path.

**Parameters:**

- `path` (str): File system path to write to.
- `data` (bytes): Data to write.

**Operation:**

1. Opens the file in binary write mode.
2. Writes the data bytes.

**Example Usage:**

```python
write_file('.git/refs/heads/master', (commit_sha1 + '\n').encode())
```

---

## Commit Creation Flow Summary

The commit creation process in `pygit` involves a chain of operations starting from the current index state to finalizing a commit object and updating the repository's `master` branch reference:

```
+-------------------------+
|  Working Directory Files |
+------------+------------+
             |
       git add (index)
             |
+------------v------------+
|       Git Index          |  <- tracked files snapshot
+------------+------------+
             |
      write_tree()
             |
       Tree Object SHA-1
             |
   +---------v---------+
   |   commit()        |
   |  - tree: <sha1>   |
   |  - parent: <sha1> |
   |  - author info    |
   |  - message        |
   +---------+---------+
             |
      Commit Object SHA-1
             |
    Update master branch ref
```

This chain ensures a consistent snapshot of the project files is captured and committed with metadata, forming the basis of the repository's history.

---

# End of `commit.md` documentation file content.