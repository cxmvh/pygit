# index_management.md

## Overview

This document provides detailed information on managing the Git index within the repository. The Git index (also known as the staging area) is a critical component that tracks the files slated for the next commit. This section explains how to read, write, and update the index file, including adding files to the index. It fits within the broader "Repository Initialization and Setup" section of the documentation tree, bridging between repository setup and commit management. Proper handling of the index is essential to ensure accurate tracking of file changes and successful commit operations.

---

## Function Documentation

### `read_index()`

**Purpose:**  
Reads the current Git index file and returns a list of `IndexEntry` objects representing the files staged in the index.

**Parameters:**  
None

**Returns:**  
- `List[IndexEntry]`: A list of index entries each containing metadata such as timestamps, device/inode numbers, file mode, SHA-1 hash, and file path.

**Operation:**  
1. Attempts to read the `.git/index` file. If not found, returns an empty list (indicating an empty index).  
2. Verifies the integrity of the index file by checking the SHA-1 checksum at the end.  
3. Parses the header to confirm the file signature (`DIRC`) and version (only version 2 supported).  
4. Iterates through the index entries, unpacking fixed-size metadata and variable-length path names.  
5. Constructs `IndexEntry` objects and appends to the entries list.  
6. Returns the list of parsed entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file, updating the staged snapshot.

**Parameters:**  
- `entries (List[IndexEntry])`: The list of entries to write to the index.

**Returns:**  
None

**Operation:**  
1. Packs each `IndexEntry` into a binary format with fixed-size fields and null-padded path strings.  
2. Packs a header containing the signature (`DIRC`), version number, and the number of entries.  
3. Concatenates all packed entries and header, then computes a SHA-1 checksum of the data.  
4. Writes the combined data and checksum to the `.git/index` file atomically.

**Example Usage:**
```python
from pygit import read_index, write_index

entries = read_index()
# Modify entries as needed, e.g., remove an entry
entries = [e for e in entries if e.path != 'unwanted_file.txt']
write_index(entries)
```

---

### `add(paths)`

**Purpose:**  
Adds one or more files specified by `paths` to the Git index, updating the staging area with their current content.

**Parameters:**  
- `paths (List[str])`: List of file paths to be added to the index.

**Returns:**  
None

**Operation:**  
1. Normalizes path separators to Unix-style `/`.  
2. Reads the current index entries, filtering out any existing entries matching the given paths (to avoid duplicates).  
3. For each specified path:  
   - Reads the file content and hashes it as a Git blob object, writing the object if necessary.  
   - Retrieves filesystem metadata (timestamps, device, inode, mode, user/group IDs, and size).  
   - Sets the flags based on path length (ensuring it fits within allowed bits).  
   - Creates a new `IndexEntry` with the collected data and appends it.  
4. Sorts all entries by path name.  
5. Writes the updated list back to the index file.

**Example Usage:**
```python
add(['src/main.py', 'README.md'])
```

---

### `IndexEntry` Structure (Referenced)

An `IndexEntry` object encapsulates metadata about a file in the Git index:

```
+----------------+----------------+----------------+----------------------------+
| ctime seconds  | ctime nanosec  | mtime seconds  | mtime nanosec              |
+----------------+----------------+----------------+----------------------------+
| dev            | ino            | mode           | uid                        |
+----------------+----------------+----------------+----------------------------+
| gid            | size           | sha1 (20 bytes)                        | flags (2 bytes) |
+----------------+----------------+----------------+----------------------------+
| path (variable length, null-terminated, padded to 8-byte alignment)             |
+----------------------------------------------------------------------------------+
```

---

### ASCII Diagram: Index File Structure

```
.git/index file layout:

+---------------------------+
| Header                    |
|  - Signature: "DIRC"      |
|  - Version: 2             |
|  - Number of entries (N)  |
+---------------------------+
| Entry 1                   |
|  - File metadata          |
|  - Path name (null-terminated & padded) |
+---------------------------+
| Entry 2                   |
|  ...                      |
+---------------------------+
| ...                       |
+---------------------------+
| Entry N                   |
+---------------------------+
| SHA-1 checksum of above   |
+---------------------------+
```

---

## Summary

The Git index is the staging area representing the next commit snapshot. This module provides essential tools to read the current index, add new files to it, and write the updated state back to disk. These functions ensure the index remains consistent and accurately reflects the files staged for commit.

By manipulating the index through these APIs, higher-level commands such as `git add` and `git commit` can function correctly, enabling a reliable Git workflow.

---

# Appendix: Related Functions (for context)

- `hash_object(data, obj_type, write=True)`: Hashes file data as a Git object and stores it. Used internally by `add()` to store blobs.  
- `write_file(path, data)`: Helper function to write byte data to a file, used by `write_index()`.  
- `read_file(path)`: Reads file content as bytes, used by `add()` and `read_index()`.  

These functions underpin the index management by handling the low-level file and object operations necessary for Git functionality.