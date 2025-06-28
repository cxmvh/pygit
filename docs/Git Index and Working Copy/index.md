# Git Index Handling in pygit

This document details the reading, writing, and manipulation of the Git index within the pygit repository. The Git index is a critical component in Git workflows, serving as the staging area between the working directory and the repository history. This file fits into the broader "Index and Objects" section of the pygit documentation, bridging file operations with commit and diff functionalities by managing the index state accurately.

---

## Overview

The Git index (also known as the staging area) is a binary file that stores information about files staged for the next commit. It contains metadata such as file timestamps, modes, SHA-1 hashes of contents, and paths. This file explains how pygit reads this index file into memory as `IndexEntry` objects, modifies them, and writes them back to disk. Functions here facilitate:

- Parsing the `.git/index` file format safely with checksum verification.
- Writing staged entries back into the index file atomically.
- Adding new files to the index by computing their blob hashes and metadata.
- Reading the index entries for status and diff operations.

Together, these enable pygit to manage the staging area effectively, supporting commands like `add`, `diff`, and `commit`.


---

## Important Functions

### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects representing staged files.

- **Purpose:**  
  Load the current state of the index from `.git/index` into memory for manipulation or inspection.

- **Operation:**  
  1. Attempts to read the `.git/index` file; returns an empty list if not present.  
  2. Verifies the SHA-1 checksum at the end of the file to ensure integrity.  
  3. Parses the header to confirm the signature (`DIRC`) and version (expects 2).  
  4. Iterates over the index entries, each consisting of fixed-length metadata fields and a variable-length path string, padding to 8-byte alignment.  
  5. Creates and collects `IndexEntry` instances containing file metadata and SHA-1 hashes.  
  6. Returns the complete list of entries.

- **Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(f"{entry.mode:o} {entry.sha1.hex()} {entry.path}")
```

---

### `write_index(entries)`

Writes a list of `IndexEntry` objects back to the `.git/index` file.

- **Purpose:**  
  Persist changes to the index, such as after adding or removing files from staging.

- **Operation:**  
  1. Packs each `IndexEntry` into the binary format according to the Git index specification: fixed fields + path + padding.  
  2. Constructs the index header with signature, version, and entry count.  
  3. Concatenates all packed entries and appends a SHA-1 checksum of the entire content (excluding the checksum itself).  
  4. Writes the final byte sequence to `.git/index`.

- **Example Usage:**

```python
entries = read_index()
# Modify entries as needed...
write_index(entries)
```

---

### `add(paths)`

Add one or more file paths to the index, updating or creating entries for those files.

- **Purpose:**  
  Stage files by computing their blob hashes and incorporating their metadata into the index.

- **Parameters:**  
  - `paths`: List of file paths (strings) to add.

- **Operation:**  
  1. Normalize paths to use forward slashes.  
  2. Read existing index entries, filtering out entries for the specified paths (to be replaced).  
  3. For each path:  
     - Read file contents and compute the blob SHA-1 hash without writing it again if unnecessary.  
     - Gather file metadata (timestamps, mode, size, etc.).  
     - Create a new `IndexEntry` with these details.  
  4. Append new entries and sort all by path.  
  5. Write updated entries back to the index file.

- **Example Usage:**

```python
add(['README.md', 'src/main.py'])
```

---

### `read_object(sha1_prefix)`

Reads a Git object by SHA-1 prefix, returning its type and raw data.

- **Purpose:**  
  Retrieve and decode compressed Git objects (blobs, trees, commits) from the object store for index and diff operations.

- **Parameters:**  
  - `sha1_prefix`: Partial or complete SHA-1 hex string identifying the object.

- **Operation:**  
  1. Locate the full object file path from the prefix.  
  2. Read and decompress the object data.  
  3. Extract the object header (type and size) and verify size correctness.  
  4. Return a tuple `(object_type, data)`.

- **Example Usage:**

```python
obj_type, data = read_object('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391')
print(f"Object type: {obj_type}, Data length: {len(data)}")
```

---

### `diff()`

Shows the unified diff of files changed between the index and the working copy.

- **Purpose:**  
  Provide a textual diff output highlighting unstaged changes compared to the staged versions.

- **Operation:**  
  1. Obtain the list of changed files by comparing the index and working directory status.  
  2. For each changed file:  
     - Read the staged blob data from the index.  
     - Read the current working copy file.  
     - Compute a unified diff of the two versions.  
     - Print the diff to stdout, separated by lines for multiple files.

- **Example Usage:**

```python
diff()
```

- **Sample Output (conceptual):**

```
--- README.md (index)
+++ README.md (working copy)
@@ -1,3 +1,4 @@
 Line 1
 Line 2
+New line added
 Line 3
----------------------------------------------------------------------
```

---

### `IndexEntry` Structure (Conceptual)

Each entry in the Git index has the following key fields:

```
+------------------------------------------------------------+
| ctime_s | ctime_n | mtime_s | mtime_n | dev | ino | mode   |
| uid     | gid     | size    | sha1 (20 bytes) | flags       |
| path (variable length, null-terminated, padded to 8 bytes) |
+------------------------------------------------------------+
```

- `ctime_s`, `ctime_n`: Creation time seconds and nanoseconds.  
- `mtime_s`, `mtime_n`: Modification time seconds and nanoseconds.  
- `dev`, `ino`: Device and inode numbers for file uniqueness.  
- `mode`: File mode (permissions and type).  
- `uid`, `gid`: User and group IDs.  
- `size`: File size in bytes.  
- `sha1`: SHA-1 hash of the file contents (blob).  
- `flags`: Includes path length and stage information.  
- `path`: File path relative to the repository root.

---

## ASCII Diagram: Index File Layout

```
+----------------------------+
| Header (12 bytes)           |
| - Signature "DIRC" (4 bytes)|
| - Version (4 bytes)         |
| - Number of entries (4 bytes)|
+----------------------------+
| Entry 1                     |
|  - Fixed-length fields (62 bytes)|
|  - Path (variable length)   |
|  - Padding to 8-byte boundary|
+----------------------------+
| Entry 2                     |
|  ...                       |
+----------------------------+
| ...                        |
+----------------------------+
| SHA-1 checksum (20 bytes)  |
+----------------------------+
```

---

## Summary

The index handling in pygit implements the core mechanics of the Git staging area, allowing for:

- Reliable loading and validation of the index file.  
- Safe and correct writing of entries reflecting staged changes.  
- Adding files to the index with precise metadata and content hashing.  
- Integration with object reading and diffing for status reporting.

This ensures that pygit maintains a consistent and accurate staging area, essential for the correctness of commit and diff operations.

---

# Appendix: Example `add` to `commit` Workflow Using Index Functions

```python
# Stage files
add(['example.txt', 'src/module.py'])

# Show diff of staged vs working copy
diff()

# Commit staged changes
commit('Add example.txt and update module.py')
```

This sequence showcases the interaction between index reading/writing, object management, and commit creation rooted in the index file operations described here.