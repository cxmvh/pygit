# index_and_commit.md

# Git Index Handling, Tree Writing, and Commit Creation

---

## Overview

This document details the processes involved in manipulating the Git index, adding files to it, writing tree objects from the index, and creating commit objects to record repository changes. It is part of the **Repository Initialization and Core Objects** section, complementing other files on repository setup and core Git object management. Understanding these functions is crucial for grasping how Git internally stages changes, represents directory snapshots, and records project history.

---

## Function Documentation

---

### `add(paths)`

**Purpose:**  
Add one or more file paths to the Git index, preparing them for committing. This function reads file contents, creates blob objects, and updates the index entries accordingly.

**Parameters:**  
- `paths` (list of str): List of file paths to add to the index.

**Operation Steps:**  
1. Normalize paths to use forward slashes.  
2. Read the current index entries and filter out entries matching the given paths to avoid duplicates.  
3. For each path:  
   - Read file content and hash it as a blob object.  
   - Gather file metadata (ctime, mtime, device, inode, mode, uid, gid, size).  
   - Create an `IndexEntry` object with the metadata, blob SHA-1, and path length flags.  
4. Append new entries and sort all entries by path.  
5. Write the updated entries back to the index file.

**Usage Example:**  
```python
# Adding files to the index
files_to_add = ['src/main.py', 'README.md']
add(files_to_add)
```

---

### `write_index(entries)`

**Purpose:**  
Serialize and write a list of `IndexEntry` objects to the Git index file (`.git/index`).

**Parameters:**  
- `entries` (list of `IndexEntry`): Index entries to write.

**Operation Steps:**  
1. For each `IndexEntry`, pack its metadata fields using struct with network byte order.  
2. Append the file path and pad the entry to an 8-byte boundary with null bytes.  
3. Concatenate all packed entries.  
4. Write a header containing signature `'DIRC'`, version number `2`, and the entry count.  
5. Append a SHA-1 checksum of all preceding data for integrity.  
6. Write the entire data to `.git/index`.

**Usage Example:**  
```python
# Assuming 'entries' is a list of IndexEntry objects
write_index(entries)
```

---

### `write_tree()`

**Purpose:**  
Create a tree object representing the current state of the Git index and write it to the object store. This tree object serves as a snapshot of the directory contents at commit time.

**Parameters:**  
None.

**Operation Steps:**  
1. Read all index entries.  
2. For each entry, construct a tree entry with mode, path, and SHA-1 of the blob.  
3. Concatenate all tree entries and hash them as a tree object.  
4. Write the tree object to the Git object database.  
5. Return the SHA-1 hash of the new tree object.

**Usage Example:**  
```python
tree_sha1 = write_tree()
print(f"Created tree object with SHA-1: {tree_sha1}")
```

---

### `commit(message, author=None)`

**Purpose:**  
Commit the current index state to the `master` branch with a given commit message. Creates a commit object referencing the tree and previous commit(s).

**Parameters:**  
- `message` (str): Commit message describing the changes.  
- `author` (str, optional): Author information in format `"Name <email>"`. If not provided, reads from environment variables.

**Operation Steps:**  
1. Generate a tree object from the current index (`write_tree`).  
2. Retrieve the current `master` branch commit hash (`get_local_master_hash`).  
3. Compose author and committer metadata with timestamps and timezone offset.  
4. Assemble commit object content including tree, parent(s), author, committer, and message.  
5. Hash and write the commit object to the object store.  
6. Update the `refs/heads/master` ref to point to the new commit hash.  
7. Print confirmation and return the commit SHA-1.

**Usage Example:**  
```python
commit_hash = commit("Initial commit")
print(f"Committed with hash: {commit_hash}")
```

---

### `get_local_master_hash()`

**Purpose:**  
Retrieve the SHA-1 hash of the latest commit on the local `master` branch.

**Parameters:**  
None.

**Operation Steps:**  
1. Read the contents of `.git/refs/heads/master`.  
2. Return the commit hash string or `None` if the ref does not exist.

**Usage Example:**  
```python
current_master = get_local_master_hash()
if current_master:
    print(f"Current master commit: {current_master}")
else:
    print("No commits on master branch yet.")
```

---

### `read_index()`

**Purpose:**  
Read and parse the Git index file, returning the list of index entries.

**Parameters:**  
None.

**Operation Steps:**  
1. Read raw index file bytes.  
2. Verify the SHA-1 checksum matches the data.  
3. Parse the header, confirming signature and version.  
4. Iterate over entries, unpacking metadata and paths.  
5. Return a list of `IndexEntry` objects.

**Usage Example:**  
```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} - SHA1: {entry.sha1.hex()}")
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Hash data as a Git object of a specified type, optionally writing it to the object store.

**Parameters:**  
- `data` (bytes): The content to hash.  
- `obj_type` (str): Git object type (`blob`, `tree`, `commit`).  
- `write` (bool): If `True`, write the compressed object to `.git/objects`.

**Operation Steps:**  
1. Construct object header with type and length.  
2. Compute SHA-1 hash of `header + \0 + data`.  
3. If `write` is `True`, compress and write the object to the appropriate `.git/objects/xx/yyyy...` path.  
4. Return the hex SHA-1 hash string.

**Usage Example:**  
```python
blob_sha1 = hash_object(b"Hello, Git!", "blob")
print(f"Blob object stored with SHA-1: {blob_sha1}")
```

---

### `read_file(path)`

**Purpose:**  
Read the raw bytes of a file from disk.

**Parameters:**  
- `path` (str): Path to the file.

**Operation Steps:**  
1. Open the file in binary mode.  
2. Read and return its contents.

**Usage Example:**  
```python
content = read_file("README.md")
print(content.decode())
```

---

### `write_file(path, data)`

**Purpose:**  
Write bytes data to a file at the given path, creating or overwriting it.

**Parameters:**  
- `path` (str): Destination file path.  
- `data` (bytes): Data to write.

**Operation Steps:**  
1. Open file in binary write mode.  
2. Write data bytes.

**Usage Example:**  
```python
write_file(".git/HEAD", b"ref: refs/heads/master")
```

---

### ASCII Diagram: Git Object Flow for Commit Creation

```
[Working Directory Files]
          |
          v  (add)
    [Git Index Entries]
          |
          v  (write_tree)
     [Tree Object SHA-1]
          |
          v  (commit)
    [Commit Object SHA-1]
          |
          v
   [refs/heads/master updated]
```

This diagram shows how files are added to the index, which is then written as a tree object. The commit object references this tree and updates the master branch.

---

## Summary

This document covers the critical steps in staging files (`add`), writing the tree object (`write_tree`), and creating a commit (`commit`). It includes low-level file and index manipulation, hashing of Git objects, and handling repository references. These functions form the backbone of Git's version control mechanism, enabling users to record and track changes efficiently.