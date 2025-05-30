# index.md

# Reading and Writing the Git Index, Adding Files to the Index

---

## Overview

This document provides comprehensive technical reference material on managing the Git index within the `pygit` project. The Git index, often referred to as the staging area, is a critical data structure that tracks the state of files between the working directory and the repository's commit history. This file covers the reading and writing of the Git index file (`.git/index`), adding files to the index, and managing index entries.

Located under the **Working Copy and Index Management** section of the documentation tree, this file plays a pivotal role in understanding how the `pygit` tool tracks changes, stages files, and prepares commits. It complements related documentation on status reporting, diffing, and committing changes, linking low-level index operations with higher-level Git workflows such as `pygit.commit`.

---

## Functions

### 1. `read_index()`

**Purpose:**  
Reads the Git index file from `.git/index` and returns a list of `IndexEntry` objects representing the current state of the staging area.

**Parameters:**  
None

**Returns:**  
`List[IndexEntry]` — a list of index entries, each containing metadata and SHA-1 hashes of staged files.

**Operation Details:**  
- Reads the raw index file bytes.  
- Verifies the SHA-1 checksum at the end of the file for integrity.  
- Parses the index header (signature, version, number of entries).  
- Iteratively unpacks each index entry, decoding the path and file metadata.  
- Returns a list of parsed `IndexEntry` objects.

**Usage Example:**  
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### 2. `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file `.git/index`, updating the staging area.

**Parameters:**  
- `entries` (`List[IndexEntry]`): The index entries to write.

**Operation Details:**  
- Serializes each index entry into a packed binary format following Git's index file specification.  
- Pads entries to 8-byte alignment.  
- Writes a header containing signature, version, and number of entries.  
- Appends a SHA-1 checksum of all preceding data to ensure integrity.  
- Writes the complete data to `.git/index`.

**Usage Example:**  
```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### 3. `add(paths)`

**Purpose:**  
Adds the specified file paths to the Git index, updating or creating index entries as needed.

**Parameters:**  
- `paths` (`List[str]`): List of file paths to add to the index.

**Operation Details:**  
- Normalizes paths to use forward slashes.  
- Reads current index entries and filters out any entries matching paths to be added (to avoid duplicates).  
- For each path:  
  - Reads file contents and creates a Git blob object (via `hash_object`).  
  - Gathers file metadata (timestamps, device, inode, mode, uid, gid, size).  
  - Creates a new `IndexEntry` with the SHA-1 hash of the blob and metadata.  
- Appends new entries and sorts all entries by path.  
- Writes updated entries back to the index.

**Usage Example:**  
```python
add(['README.md', 'src/main.py'])
```

---

### 4. `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries and writes it to the object store. Returns the SHA-1 hash of the created tree.

**Parameters:**  
None

**Returns:**  
`str` — SHA-1 hex string of the written tree object.

**Operation Details:**  
- Reads all index entries.  
- For each entry, constructs a tree entry consisting of the file mode and path, followed by the SHA-1 of the blob.  
- Concatenates all tree entries and writes them as a Git tree object via `hash_object`.  
- Returns the SHA-1 hash of the tree.

**Usage Example:**  
```python
tree_sha1 = write_tree()
print(f"Tree SHA-1: {tree_sha1}")
```

---

### 5. `ls_files(details=False)`

**Purpose:**  
Lists files currently staged in the Git index.

**Parameters:**  
- `details` (`bool`, optional): If True, prints detailed info including mode, SHA-1, and stage number; otherwise, prints only paths.

**Operation Details:**  
- Reads the index entries.  
- For each entry, prints the path or detailed information depending on the `details` flag.

**Usage Example:**  
```python
# Simple listing
ls_files()

# Detailed listing
ls_files(details=True)
```

---

### 6. `get_status()`

**Purpose:**  
Determines the status of the working directory relative to the index, identifying changed, new, and deleted files.

**Parameters:**  
None

**Returns:**  
Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`.

**Operation Details:**  
- Walks the working directory recursively, excluding `.git`.  
- Collects all file paths currently present.  
- Reads entries from the index.  
- Compares file contents of files present in both working directory and index by hashing blobs.  
- Identifies files that are changed, newly added, or deleted relative to the index.

**Usage Example:**  
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### 7. `status()`

**Purpose:**  
Prints a human-readable summary of the working directory status relative to the index.

**Parameters:**  
None

**Operation Details:**  
- Calls `get_status()` to retrieve changed, new, and deleted files.  
- Prints lists under "changed files:", "new files:", and "deleted files:" headings, if any.

**Usage Example:**  
```python
status()
```

---

## ASCII Diagram: Git Index Workflow

```
+-----------------+         +------------------+         +-----------------+
|  Working Copy   |  <--->  |      Git Index    |  <--->  |   Object Store  |
| (filesystem)    |         |  (.git/index)     |         |  (.git/objects) |
+-----------------+         +------------------+         +-----------------+
        |                             |                           |
        |  add(path)                  |                           |
        |---------------------------->                           |
        |                             |                           |
        |                             |  write_index(entries)     |
        |                             |-------------------------->|
        |                             |                           |
        |                             |      write_object(blob)   |
        |                             |-------------------------->|
        |                             |                           |
        |                             |                           |
        |                         read_index()                    |
        |<----------------------------                           |
        |                             |                           |
        |                         write_tree()                    |
        |--------------------------------------------------------->|
        |                             |                           |
        |                       commit() uses tree SHA-1         |
        |                             |                           |
```

---

## Additional Notes

- The `IndexEntry` data structure encapsulates file metadata, including creation/modification times, device and inode numbers, file permissions, user/group IDs, file size, SHA-1 of the content, and flags. This facilitates precise tracking of file states and detection of changes.

- The index file format is binary and requires strict adherence to Git's specification for compatibility.

- The `add()` function integrates tightly with `hash_object()` to store file contents as blobs in the object store before updating the index.

- This document references core supporting functions such as `hash_object()`, `write_file()`, and `read_file()`, which handle low-level file and object management.

---

# End of index.md Documentation Content