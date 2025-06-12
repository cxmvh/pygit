# commit.md - Commit Operation Documentation

---

## Overview

The `commit.md` file documents the commit operation within the pygit project, focusing on creating commit objects and updating repository references (refs). This file details how commits are created, including the construction of tree objects representing the working directory state, capturing author and committer information, and updating the `master` branch reference. It fits into the broader "Core Commands" section of the pygit documentation tree, providing essential insights into commit creation and management, which is fundamental to version control workflows in Git.

---

## Function Documentation

### `commit(message, author=None)`

**Purpose:**  
Creates a new commit object representing the current state of the repository at the time of execution, recording the commit message and author details. It writes the commit object to the Git object store, updates the `master` branch reference, and returns the SHA-1 hash of the new commit.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): The author information string (e.g., `"Name <email>"`). If not provided, it defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Generate a tree object from the current index by calling `write_tree()`.  
2. Retrieve the hash of the current `master` branch commit via `get_local_master_hash()`, to set as the commit's parent (if any).  
3. If no author is specified, extract author name and email from the environment variables.  
4. Construct author and committer timestamp strings with timezone offset.  
5. Formulate commit content lines including:  
   - The tree object hash line.  
   - The parent commit hash line (if a parent exists).  
   - Author and committer lines with timestamps.  
   - The commit message.  
6. Encode the commit data and hash it using `hash_object()` with type `'commit'`.  
7. Write the commit hash to the `.git/refs/heads/master` file to update the branch reference.  
8. Print a confirmation message and return the commit SHA-1 hash.

**Example Usage:**
```python
commit_hash = commit("Add new feature implementation")
print(f"Created commit {commit_hash} on master branch.")
```

---

### `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries representing the snapshot of the working directory for the commit. It encodes file metadata and content hashes into a tree object and writes it to the object store.

**Parameters:**  
None.

**Operation Steps:**  
1. Read the current index entries using `read_index()`.  
2. For each entry, verify it is a top-level file (no nested directories supported currently).  
3. Format each entry as `<mode> <path>\0<sha1>`, where:  
   - `mode` is the file mode in octal.  
   - `path` is the file path.  
   - `sha1` is the binary SHA-1 hash of the blob object.  
4. Concatenate all entries and hash them with `hash_object()` as a `'tree'` object.  
5. Return the SHA-1 hash of the resulting tree object.

**Example Usage:**
```python
tree_hash = write_tree()
print(f"Tree object created with hash: {tree_hash}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieves the SHA-1 hash of the current commit at the `master` branch reference.

**Parameters:**  
None.

**Operation Steps:**  
1. Read the contents of `.git/refs/heads/master`.  
2. If the reference file does not exist, return `None`.  
3. Strip whitespace and return the commit hash string.

**Example Usage:**
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits on master branch yet.")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Hashes Git object data by prepending an object-type header, compresses it, and writes it to the object store if indicated.

**Parameters:**  
- `data` (bytes): Raw data of the Git object.  
- `obj_type` (str): Object type e.g., `'commit'`, `'tree'`, `'blob'`.  
- `write` (bool): If `True`, write the compressed object data to the `.git/objects` directory.

**Operation Steps:**  
1. Create the object header `<obj_type> <len(data)>\0`.  
2. Concatenate header and data, then compute SHA-1 hash.  
3. If `write` is `True`, save the compressed object data in the `.git/objects/` directory with subdirectory and filename based on the SHA-1.  
4. Return the SHA-1 hash as a hex string.

**Example Usage:**
```python
sha1 = hash_object(b"Hello World\n", "blob")
print(f"Object stored with SHA-1: {sha1}")
```

---

### `write_file(path, data)`

**Purpose:**  
Writes raw byte data to a file at the specified path, creating or overwriting it.

**Parameters:**  
- `path` (str): File path where data will be written.  
- `data` (bytes): Byte data to write.

**Operation Steps:**  
1. Open the file at `path` in binary write mode.  
2. Write the `data` bytes to the file.  
3. Close the file.

**Example Usage:**
```python
write_file(".git/refs/heads/master", b"abcdef1234567890\n")
print("Updated master branch reference.")
```

---

## Commit Operation ASCII Diagram

```
+----------------------------+
|          commit()          |
+----------------------------+
            |
            v
+----------------------------+
|        write_tree()        |  <-- Reads index and writes tree object
+----------------------------+
            |
            v
+----------------------------+
|   get_local_master_hash()  |  <-- Reads current master commit hash
+----------------------------+
            |
            v
+----------------------------+
| Construct commit data lines |  <-- Includes tree, parent, author, committer, message
+----------------------------+
            |
            v
+----------------------------+
|      hash_object(data)     |  <-- Creates commit object and writes it
+----------------------------+
            |
            v
+----------------------------+
| update master ref file     |  <-- Writes new commit hash to .git/refs/heads/master
+----------------------------+
            |
            v
+----------------------------+
|        Return SHA-1        |
+----------------------------+
```

---

## Summary

The `commit` command in pygit encapsulates the process of snapshotting the current index state into a tree object, linking it to a parent commit if present, and writing a commit object that references this tree. It finalizes the commit by updating the `master` branch reference. This process ensures that the repository's history is properly maintained and accessible for subsequent operations such as pushes, diffs, and checkouts. Understanding these functions and their interplay is essential for developers working on or extending pygit's commit functionality.