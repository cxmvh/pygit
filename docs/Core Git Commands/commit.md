# commit.md

## Overview

This document details the process of committing changes to the local Git repository in the `pygit` project, covering the creation of commit objects, writing trees, and updating branch references. It sits within the **Core Git Commands** section of the documentation tree, closely linked with repository management, object handling, and index management. The commit operation is a fundamental part of version control, encapsulating the current state of the project and its history in a new commit object. This file explains the key functions enabling commit creation, interaction with the Git object store, and reference updates that record the commit in the local repository.

---

## Function Documentation

### commit(message, author=None)

**Purpose:**  
Create a new commit object in the local repository representing the current state of the index, attaching the specified commit message and author information. It updates the `refs/heads/master` reference to point to this new commit.

**Parameters:**  
- `message` (str): The commit message describing the changes.  
- `author` (str, optional): Author info in `"Name <email>"` format. If not provided, the function uses environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Call `write_tree()` to write the current index entries as a tree object and get its SHA-1 hash.  
2. Retrieve the current commit hash (parent) from the local `master` branch using `get_local_master_hash()`.  
3. Determine the author string and timestamp formatted with timezone offset.  
4. Construct commit content lines including tree hash, optional parent commit, author, committer (same as author), and commit message.  
5. Encode and hash the commit data using `hash_object()` to create the commit object in the object store.  
6. Update the local `refs/heads/master` file to point to the new commit’s SHA-1 hash.  
7. Print a confirmation message with the commit hash and return the hash.

**Example Usage:**

```python
commit_hash = commit("Fix bug in user authentication flow")
print(f"New commit created: {commit_hash}")
```

---

### write_tree()

**Purpose:**  
Serialize the current index entries into a Git tree object and write it to the object store, returning the tree’s SHA-1 hash.

**Operation Steps:**  
1. Read all index entries via `read_index()`.  
2. For each entry, assert it is in the top-level directory (no slashes in path).  
3. Format each entry as `<mode> <filename>\0<sha1>` bytes.  
4. Concatenate all entries and hash them as a `tree` object using `hash_object()`.  
5. Return the resulting tree SHA-1 hash.

**Example Usage:**

```python
tree_hash = write_tree()
print(f"Tree object created with hash: {tree_hash}")
```

---

### get_local_master_hash()

**Purpose:**  
Retrieve the current SHA-1 hash of the local `master` branch commit.

**Operation Steps:**  
1. Read the file `.git/refs/heads/master`.  
2. Decode and strip whitespace to return the commit hash string.  
3. Return `None` if the reference file does not exist.

**Example Usage:**

```python
master_hash = get_local_master_hash()
print(f"Current master commit hash: {master_hash}")
```

---

### hash_object(data, obj_type, write=True)

**Purpose:**  
Compute the SHA-1 hash of a Git object of a given type and optionally write it to the object store.

**Parameters:**  
- `data` (bytes): Raw data of the object.  
- `obj_type` (str): Git object type (e.g., `'commit'`, `'tree'`, `'blob'`).  
- `write` (bool): If True, write the compressed object to `.git/objects`.

**Operation Steps:**  
1. Construct the header `<obj_type> <len(data)>` followed by a null byte.  
2. Compute SHA-1 hash of header + data.  
3. If `write` is True and the object file does not exist, compress and write it to the object store path `.git/objects/xx/yyyy...` where `xx` is first two SHA-1 chars.  
4. Return the SHA-1 hash as a hex string.

**Example Usage:**

```python
data = b"example content"
obj_hash = hash_object(data, "blob")
print(f"Blob object created with hash: {obj_hash}")
```

---

### write_file(path, data)

**Purpose:**  
Write raw bytes `data` to the file at the specified filesystem `path`.

**Operation Steps:**  
1. Open the file in binary write mode.  
2. Write the data bytes.  
3. Close the file to flush contents.

**Example Usage:**

```python
write_file(".git/refs/heads/master", b"abcdef1234567890\n")
```

---

## ASCII Diagram: Commit Creation Flow

```
+------------------------+
| Current Index Entries   |           (read via read_index())
+-----------+------------+
            |
            v
+------------------------+         (write_tree)
| Tree Object Created    | <--------------------+
+-----------+------------+                      |
            |                                   |
            v                                   |
+------------------------+                      |
| Compose Commit Content |                      |
| (tree, parent, author, |                      |
|  committer, message)   |                      |
+-----------+------------+                      |
            |                                   |
            v                                   |
+------------------------+                      |
| Hash Commit Object     | -- hash_object() -->| (writes commit object to .git/objects)
+-----------+------------+                      |
            |                                   |
            v                                   |
+------------------------+                      |
| Update refs/heads/master| <----- write_file()|
+------------------------+
```

---

## Summary

The `commit.md` file documents the core process of creating a commit in `pygit`. It involves writing a tree object representing the current index, composing the commit content with metadata and parent commits, hashing and storing this commit object, and updating the `master` branch reference. Together with related functions for hashing objects and managing references, this forms the foundation for recording changes in the local Git repository.