# file_operations.md

# File Operations: Reading, Writing, and Hashing Critical for Git Functionality

---

## Overview

This document covers core file operations essential for Git functionality, focusing on reading file contents, writing data to files, and hashing objects to store them efficiently in the Git object database. These operations underpin higher-level Git features such as status reporting, committing, and diff generation. This file fits within the **Index and Working Copy Management** section of the documentation tree, supporting commands like `status`, `add`, and `commit` by providing reliable low-level file manipulation and hashing utilities.

Specifically, these functions handle:

- Reading files as bytes for hashing or object storage.
- Writing arbitrary data to files, including compressed Git objects.
- Hashing file contents into Git objects (blobs, trees, commits) with SHA-1.
- Assisting in verifying file changes by comparing object hashes.

They provide the foundation that allows the Git implementation to track file changes, store objects, and maintain repository integrity.

---

## Function Documentation

### 1. `read_file(path)`

**Purpose:**  
Reads the entire contents of a file at the specified path and returns the data as bytes.

**Parameters:**  
- `path` (str): The path to the file to read.

**Operation Details:**  
- Opens the file in binary read mode (`'rb'`).
- Reads all bytes from the file.
- Returns the byte content for further processing (e.g., hashing, diffing).

**Usage Example:**
```python
file_bytes = read_file('README.md')
print(f"Read {len(file_bytes)} bytes from README.md")
```

---

### 2. `write_file(path, data)`

**Purpose:**  
Writes raw bytes data to a file at the given path.

**Parameters:**  
- `path` (str): The file path to write to.
- `data` (bytes): The bytes content to write.

**Operation Details:**  
- Opens the file in binary write mode (`'wb'`).
- Writes the entire data buffer to the file.
- Ensures the file content exactly matches the provided data.

**Usage Example:**
```python
write_file('.git/objects/ab/cdef1234...', compressed_data)
print("Object written to disk.")
```

---

### 3. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of a Git object given its raw data and type, optionally writing the compressed object into the `.git/objects` directory.

**Parameters:**  
- `data` (bytes): Raw content of the Git object (e.g., file contents for blobs).
- `obj_type` (str): The type of Git object: `'blob'`, `'tree'`, `'commit'`, etc.
- `write` (bool, default `True`): If `True`, writes the compressed object to the object store.

**Operation Details:**  
1. Constructs the Git object header: `<obj_type> <data_length>\0`.
2. Concatenates header and data to form the full object.
3. Computes SHA-1 hash of the full object.
4. If `write` is `True`:
   - Determines path in `.git/objects/` based on SHA-1 (first two hex chars as directory, rest as filename).
   - Creates directories if needed.
   - Compresses the full object using zlib.
   - Writes compressed object to disk.
5. Returns the SHA-1 hash as a hexadecimal string.

**Usage Example:**
```python
file_data = read_file('example.txt')
sha1_hash = hash_object(file_data, 'blob')
print(f"Object stored with SHA-1: {sha1_hash}")
```

---

### ASCII Diagram: Object Storage Layout

```
.git/
 └── objects/
     ├── ab/
     │    └── cdef1234...  # Object file named by SHA-1 suffix
     ├── 12/
     │    └── 3456789a...
     └── ...              # Many directories named by first 2 SHA-1 chars
```

This structure allows efficient storage and retrieval of Git objects by their SHA-1 hash.

---

### 4. `get_status()`

**Purpose:**  
Determines the status of the working copy relative to the Git index, returning three lists of file paths categorized as changed, new, or deleted.

**Parameters:**  
None.

**Operation Details:**  
- Walks the current directory tree excluding `.git`.
- Collects all file paths present on disk.
- Reads the Git index entries and extracts paths tracked by Git.
- For paths present in both working copy and index:
  - Reads file content, hashes it, compares to stored SHA-1.
  - Files with mismatched hashes are marked as changed.
- Files present on disk but not in index are new.
- Files in index but missing on disk are deleted.

**Returns:**  
Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`

**Usage Example:**
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 5. `status()`

**Purpose:**  
Displays the status of the working copy by printing lists of changed, new, and deleted files.

**Parameters:**  
None.

**Operation Details:**  
- Calls `get_status()` to retrieve file status.
- Prints categorized lists with indentation for readability.

**Usage Example:**
```python
status()
# Output:
# changed files:
#     README.md
# new files:
#     new_script.py
# deleted files:
#     old_notes.txt
```

---

### Notes on Integration

- `read_file` and `write_file` are fundamental utilities used throughout Git operations.
- `hash_object` enables consistent content-addressable storage of file contents, trees, and commits.
- `get_status` and `status` rely on these functions to compare working directory state against the index efficiently.
- These functions together facilitate the building blocks for commands like `git add`, `git commit`, and `git status`.

---

## Summary

This documentation outlines essential file operation functions that enable Git's core capabilities of tracking file changes and storing data reliably within the `.git` directory. By reading files as bytes, writing compressed object data, and hashing those objects, these functions support the integrity and content-addressability that Git depends on.

---

# End of file_operations.md documentation content.