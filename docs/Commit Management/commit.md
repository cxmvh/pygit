# Commit Management Documentation

## Overview

This document covers the procedures and functions used for creating commit objects, writing tree objects, and managing commit metadata within the Git repository. It is part of the broader **Commit Management** section in the documentation tree, which focuses on the lifecycle of commits and their associated metadata. This file details how the current state of the repository index is captured as a commit, how commit objects are constructed and stored, and how commit metadata such as author information and timestamps are managed. Understanding these functions is fundamental for developers working on Git internals, especially for operations related to recording new repository states.

---

## Function: `commit(message, author=None)`

### Purpose

Creates a new commit object representing the current state of the repository index and updates the `master` branch reference. Returns the SHA-1 hash of the newly created commit object.

### Parameters

- `message` (str): The commit message describing the changes.
- `author` (str, optional): Author metadata in the format `"Name <email>"`. If not provided, it defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

### Description

1. Calls `write_tree()` to write the current index as a tree object and obtain its SHA-1 hash.
2. Obtains the current commit hash of the local `master` branch using `get_local_master_hash()`.
3. If no author is specified, constructs the author string from environment variables.
4. Computes the current timestamp and formats the author time string with timezone offset.
5. Constructs the commit content lines, including:
   - `tree` reference line.
   - Optional `parent` commit line if a parent exists.
   - `author` and `committer` lines with timestamps.
   - An empty line followed by the commit message.
6. Joins and encodes these lines as a byte string.
7. Calls `hash_object()` with the commit data and type `'commit'` to hash and store the commit object.
8. Writes the resulting SHA-1 hash to the appropriate `master` reference file inside `.git/refs/heads/master`.
9. Prints a confirmation message showing the commit hash.
10. Returns the commit SHA-1 hash.

### Example Usage

```python
commit_hash = commit("Initial commit")
print(f"Created commit: {commit_hash}")
```

---

## Function: `write_tree()`

### Purpose

Creates a tree object from the current Git index entries, writing the snapshot of the working directory's file hierarchy at the current point in time. Returns the SHA-1 hash of the tree object.

### Parameters

None.

### Description

1. Reads the current index entries using `read_index()`.
2. For each entry, verifies that the path does not contain subdirectory separators (only supports top-level files).
3. Formats each entry as a tree entry by combining the file mode, path, a null byte, and the SHA-1 hash of the blob.
4. Concatenates all tree entries into a single byte string.
5. Calls `hash_object()` with the concatenated tree entries and object type `'tree'` to write and hash the tree object.
6. Returns the SHA-1 hash of the created tree object.

### Example Usage

```python
tree_hash = write_tree()
print(f"Tree object created with hash: {tree_hash}")
```

---

## Function: `get_local_master_hash()`

### Purpose

Retrieves the SHA-1 hash of the current commit pointed to by the local `master` branch.

### Parameters

None.

### Description

1. Reads the content of `.git/refs/heads/master`.
2. Returns the decoded and stripped SHA-1 hash string.
3. If the reference file does not exist (e.g., no commits yet), returns `None`.

### Example Usage

```python
master_commit = get_local_master_hash()
if master_commit:
    print(f"Current master commit: {master_commit}")
else:
    print("No commits on master branch yet.")
```

---

## Function: `hash_object(data, obj_type, write=True)`

### Purpose

Calculates the SHA-1 hash of an object of a given type, optionally writes the object to the Git object store, and returns the SHA-1 hash.

### Parameters

- `data` (bytes): The raw content of the object.
- `obj_type` (str): The type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (bool, optional): Whether to write the object to the `.git/objects` directory (default is `True`).

### Description

1. Creates a header string with the format `"{obj_type} {len(data)}"`.
2. Concatenates the header, a null byte, and the data to form the full object data.
3. Computes the SHA-1 hash of the full object data.
4. If `write` is `True`:
   - Constructs the path in `.git/objects` based on the SHA-1.
   - Creates the directory if it does not exist.
   - Compresses the full data using zlib and writes it to the object file.
5. Returns the SHA-1 hash as a hexadecimal string.

### Example Usage

```python
blob_data = b"Hello, world!\n"
blob_hash = hash_object(blob_data, 'blob')
print(f"Blob object hash: {blob_hash}")
```

---

## Function: `write_file(path, data)`

### Purpose

Writes raw byte data to a file at the specified filesystem path.

### Parameters

- `path` (str): The filesystem path where the data should be written.
- `data` (bytes): The data to write to the file.

### Description

1. Opens the file at `path` in binary write mode.
2. Writes the byte data to the file.
3. Closes the file.

### Example Usage

```python
write_file('.git/refs/heads/master', b'abc123...\n')
```

---

## ASCII Diagram: Commit Object Structure

```
+-------------------------------+
| commit                        |
|                               |
| tree <tree_hash>              |
| parent <parent_commit_hash>   |  <-- Optional (if exists)
| author <author_name> <email> <timestamp> <timezone>
| committer <committer_name> <email> <timestamp> <timezone>
|                               |
| <commit message>              |
+-------------------------------+
```

---

## Summary

The `commit.md` documentation highlights the key functions involved in creating commit objects in the Git repository, including:

- Capturing the current index snapshot as a tree object (`write_tree`).
- Composing commit metadata and writing commit objects (`commit`).
- Managing object storage and hashing (`hash_object`).
- Reading and updating branch references (`get_local_master_hash`, `write_file`).

These functions collectively enable the recording of repository history in the form of commits, a core aspect of Git's functionality.

---

For further information on related operations such as index management, object handling, and pushing commits to remote repositories, consult the following files in the documentation tree:

- [Index Management](../index.md)
- [Object Handling](../objects.md)
- [Push and Remote Operations](../push.md)