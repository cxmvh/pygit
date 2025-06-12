# index_and_commit.md

## Overview

This document explains the management of the Git index (also known as the staging area), the process of staging files, and committing changes to a Git repository. The index is a critical component that tracks the state of files between the working directory and the repository history. Understanding how to manipulate the index and perform commits is essential for controlling the snapshot history of a project. This file fits within the broader documentation tree under "Repository Initialization and Core Concepts," providing foundational knowledge that connects repository setup, index handling, and commit creation.

---

## Function Documentation

### `add(paths)`

**Purpose:**  
Stage one or more files by adding them to the Git index. This updates the index entries to reflect the current content of the specified files, preparing them for the next commit.

**Parameters:**  
- `paths` (list of str): List of file paths to stage. Paths are normalized to use forward slashes.

**Operation Steps:**  
1. Normalize the file paths to use Unix-style separators (`/`).  
2. Read the current index entries.  
3. Filter out any existing index entries whose paths match those being added (to avoid duplicates).  
4. For each path to add:  
   - Read the file contents and hash it as a 'blob' object in the Git object store.  
   - Gather file metadata (timestamps, device, inode, mode, uid, gid, size).  
   - Create a new `IndexEntry` with the metadata, SHA-1 hash, and path.  
5. Append new entries to the filtered list.  
6. Sort all entries by their path lexicographically.  
7. Write the updated list of entries back to the index file.

**Example Usage:**
```python
add(['src/main.py', 'README.md'])
```

---

### `write_index(entries)`

**Purpose:**  
Serialize and write a list of `IndexEntry` objects to the Git index file (`.git/index`), updating the staging area.

**Parameters:**  
- `entries` (list of `IndexEntry`): List of index entries to write.

**Operation Steps:**  
1. For each index entry, pack its fields into a fixed binary structure.  
2. Append the file path encoded as bytes, padded to align on an 8-byte boundary.  
3. Concatenate all packed entries.  
4. Prepend a header containing the signature `"DIRC"`, version number, and number of entries.  
5. Calculate a SHA-1 checksum of the entire index data (excluding the checksum itself).  
6. Write the index data followed by the checksum to `.git/index`.

**Example Usage:**
```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### `commit(message, author=None)`

**Purpose:**  
Create a new commit object from the current state of the index and update the local master branch reference to point to the new commit.

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author identity in the format `"Name <email>"`. Defaults to environment variables `GIT_AUTHOR_NAME` and `GIT_AUTHOR_EMAIL`.

**Operation Steps:**  
1. Write a tree object representing the current index state (`write_tree()`).  
2. Retrieve the current commit hash of the local master branch (`get_local_master_hash()`).  
3. Prepare author and committer information, including timestamp and timezone offset.  
4. Compose the commit object content including tree, parent(s), author, committer, and message.  
5. Hash and write the commit object to the object store.  
6. Update `.git/refs/heads/master` to point to the new commit hash.  
7. Print confirmation of the commit.

**Example Usage:**
```python
commit("Fix bug in data processing module")
```

---

### `write_tree()`

**Purpose:**  
Create a Git tree object from the current index entries, representing a snapshot of the repository at the root directory level.

**Returns:**  
- SHA-1 hash of the written tree object as a hex string.

**Operation Steps:**  
1. Read all index entries.  
2. For each entry, format mode and path as a byte string, append a null byte, then append the SHA-1 hash of the blob object.  
3. Concatenate all such entries to form the tree object content.  
4. Hash and write the tree object to the object store.  
5. Return the resulting SHA-1 hash.

**Note:**  
Currently supports only a flat directory (no nested trees).

**Example Usage:**
```python
tree_hash = write_tree()
print(f"Tree object created: {tree_hash}")
```

---

### `read_index()`

**Purpose:**  
Read and parse the `.git/index` file, returning a list of `IndexEntry` objects representing the current staging state.

**Returns:**  
- List of `IndexEntry` instances.

**Operation Steps:**  
1. Attempt to read `.git/index`. Return an empty list if not found.  
2. Verify checksum at the end of the index file matches the SHA-1 digest of the preceding data.  
3. Parse the header for signature, version, and number of entries.  
4. Iteratively unpack each entry's fixed-length fields and corresponding file path.  
5. Return the list of entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the current commit hash pointed to by the local master branch reference.

**Returns:**  
- SHA-1 hash string of the latest commit on master, or `None` if no commits exist.

**Operation Steps:**  
1. Read the contents of `.git/refs/heads/master`.  
2. Return the stripped content as a SHA-1 string.  
3. Return `None` if the reference does not exist.

**Example Usage:**
```python
current_commit = get_local_master_hash()
print(f"Local master commit: {current_commit}")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Generate a SHA-1 hash for Git objects (blob, tree, commit) and optionally write the compressed object to the Git object store.

**Parameters:**  
- `data` (bytes): Raw object data.  
- `obj_type` (str): Type of object ('blob', 'tree', 'commit').  
- `write` (bool): Whether to write the object to the object store.

**Returns:**  
- Hex string SHA-1 of the object.

**Operation Steps:**  
1. Construct object header: `"<type> <size>\0"`.  
2. Concatenate header and data.  
3. Compute SHA-1 hash of the combined data.  
4. If `write` is `True`, compress and write the object to `.git/objects/xx/yyyy...`.  
5. Return the SHA-1 hash.

**Example Usage:**
```python
blob_sha = hash_object(b"Hello World\n", "blob")
print(f"Blob object SHA-1: {blob_sha}")
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file as bytes.

**Parameters:**  
- `path` (str): File system path.

**Returns:**  
- File contents as bytes.

**Example Usage:**
```python
data = read_file('README.md')
print(data.decode())
```

---

### `write_file(path, data)`

**Purpose:**  
Write bytes data to the specified file path.

**Parameters:**  
- `path` (str): Target file path.  
- `data` (bytes): Data to write.

**Example Usage:**
```python
write_file('.git/HEAD', b'ref: refs/heads/master')
```

---

## ASCII Diagram: Git Index and Commit Workflow

```
+----------------+          +------------------+          +-----------------+
| Working Copy   |   add    | Git Index        |  commit  | Commit Object   |
| (Modified files)+-------->+ (Staging area)   +--------->+ (Repository     |
|                |          |                  |          | history snapshot)|
+----------------+          +------------------+          +-----------------+
        |                          |                             |
        |                          |                             |
        |                          |                             |
        |                          |                             v
        |                          |                    +----------------+
        |                          |                    | .git/refs/heads|
        |                          |                    | /master        |
        |                          |                    +----------------+
        |                          |                             |
        |                          +-----------------------------+
        |                                                        Update
        v
    Read/Modify files
```

**Explanation:**  
- The user modifies files in the working copy.  
- Running `add()` stages changes by updating the Git index.  
- Running `commit()` creates a commit object from the index snapshot, updating the repository’s history reference (e.g., master branch).  

---

## Summary

This documentation covered the core concepts and operations for managing Git's staging area and creating commits programmatically. The key functions include `add()`, which stages file changes; `write_index()`, which persists the index; `write_tree()`, which serializes the index snapshot into a tree object; and `commit()`, which creates and records a commit object referencing the snapshot and history. Together, these operations enable controlled and verifiable versioning of project files within a Git repository.