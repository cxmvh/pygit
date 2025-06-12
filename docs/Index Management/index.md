# index.md - Git Index File Handling Documentation

---

## Overview

The `index.md` documentation file focuses on the handling of the Git index file, a critical component in Git's workflow. The index acts as a staging area that tracks the state of files as they are prepared for commit. This file documents the key functions involved in reading from and writing to the Git index file, primarily the `read_index` and `write_index` functions, along with related utilities that manage file tracking and update the index accordingly.

Situated within the broader "Index Management" section of the documentation tree, this file provides essential insights into how the index operates and integrates with commit operations (`pygit.commit`) and status checks (`pygit.status`). Understanding these functions is fundamental for developers working on Git internals or implementing Git-compatible tooling.

---

## Function Documentation

### 1. `read_index()`

**Purpose:**  
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the current state of tracked files.

**Parameters:**  
None.

**Returns:**  
- `List[IndexEntry]`: A list of index entries detailing file metadata and SHA-1 hashes.

**Preconditions:**  
- The `.git/index` file must exist and be a valid Git index file conforming to version 2 format.

**Operation Details:**  
- Opens and reads the binary index file.
- Validates file integrity by checking the SHA-1 checksum.
- Parses the header to verify the signature (`DIRC`) and version (expected version 2).
- Iteratively unpacks fixed-size entry fields and extracts file paths terminated by null bytes.
- Handles padding to align entries to an 8-byte boundary.
- Constructs and accumulates `IndexEntry` objects for each entry found.

**Usage Example:**

```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### 2. `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects to the Git index file, updating the `.git/index` file to reflect changes in the staging area.

**Parameters:**  
- `entries (List[IndexEntry])`: List of index entries to write.

**Returns:**  
None.

**Preconditions:**  
- The `entries` list must be correctly populated with valid `IndexEntry` instances.
- Each entry must have accurate metadata and SHA-1 hashes.

**Operation Details:**  
- For each entry, packs fixed metadata fields and appends the file path with null termination.
- Applies padding to ensure 8-byte alignment for each entry.
- Constructs the index header with signature, version, and entry count.
- Concatenates all packed entries with the header.
- Appends a SHA-1 checksum of the entire data (excluding checksum itself) for validation.
- Writes the complete binary data to `.git/index`.

**Usage Example:**

```python
from pygit import read_index, write_index, IndexEntry

# Modify or create entries
entries = read_index()
# Assume entries modified or new entries created here
write_index(entries)
```

---

### 3. `add(paths)`

**Purpose:**  
Adds the specified file paths to the Git index, updating or creating index entries for these files.

**Parameters:**  
- `paths (List[str])`: List of file paths to add to the index.

**Returns:**  
None.

**Preconditions:**  
- Paths must point to files present in the working directory.
- Files must be accessible and readable.

**Operation Details:**  
- Normalizes paths to use forward slashes.
- Reads current index entries and filters out entries matching the paths to be added.
- For each new path:
  - Reads file contents and computes SHA-1 hash as a Git blob object.
  - Retrieves file metadata (ctime, mtime, device, inode, mode, uid, gid, size).
  - Creates a new `IndexEntry` with gathered information and the computed SHA-1.
- Aggregates and sorts entries by path.
- Writes updated list back to the index using `write_index`.

**Usage Example:**

```python
add(['src/main.py', 'README.md'])
```

---

### 4. `read_file(path)`

**Purpose:**  
Reads the contents of the specified file as bytes.

**Parameters:**  
- `path (str)`: File path to read.

**Returns:**  
- `bytes`: Raw file data.

**Usage Example:**

```python
data = read_file('README.md')
print(data.decode())
```

---

### 5. `write_file(path, data)`

**Purpose:**  
Writes byte data to the specified file path.

**Parameters:**  
- `path (str)`: Destination file path.
- `data (bytes)`: Data to write.

**Returns:**  
None.

**Usage Example:**

```python
write_file('.git/index', binary_index_data)
```

---

## ASCII Diagram: Git Index File Structure

```
+-------------------------------+
|          Index Header          |
|  Signature (4 bytes) "DIRC"   |
|  Version (4 bytes, e.g., 2)   |
|  Number of entries (4 bytes)  |
+-------------------------------+
|          Index Entries         |
|  +-------------------------+  |
|  | Entry 1                 |  |
|  | - ctime_s, ctime_n      |  |
|  | - mtime_s, mtime_n      |  |
|  | - dev, ino              |  |
|  | - mode, uid, gid        |  |
|  | - size                  |  |
|  | - SHA-1 (20 bytes)      |  |
|  | - flags                 |  |
|  | - path (null terminated)|  |
|  | - padding for alignment |  |
|  +-------------------------+  |
|  | Entry 2                 |  |
|  | ...                     |  |
|  +-------------------------+  |
|           ...                 |
+-------------------------------+
|       SHA-1 checksum          |
|       (20 bytes)              |
+-------------------------------+
```

Each entry contains comprehensive metadata about the file tracked, allowing Git to efficiently manage file changes and staging.

---

## Integration Notes

- The `read_index` and `write_index` functions are heavily used by other core operations such as `commit` (to prepare the tree object from the index) and `status` (to compare the working directory against the index).
- The index file format is defined precisely to enable consistent and fast operations within Git.
- Index entries are sorted by path to optimize lookups.
- Checksum verification ensures index integrity before use.

---

# End of index.md documentation content.