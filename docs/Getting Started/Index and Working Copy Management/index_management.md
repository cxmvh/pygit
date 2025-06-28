# index_management.md

# Overview

This document provides a comprehensive explanation of managing the Git index within the pygit project. It covers the processes involved in reading the existing Git index file, writing changes back to it, and adding files to the index to prepare them for commits. The Git index, also known as the staging area, is a critical component in Git’s architecture, as it holds metadata about files that are to be included in the next commit. Understanding how to read and write this index enables advanced manipulation of the repository state, facilitating commit creation and status tracking.

This file belongs to the "Index and Working Copy Management" section of the pygit documentation tree, which focuses on handling the Git index and the working directory. It complements other documents like `read_index.md`, `write_index.md`, and `add.md` by providing integrated knowledge and code flows related to index handling, and it is closely associated with commit operations (`pygit.commit`).

---

# Function Documentation

## `read_index()`

### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the files currently staged in the index. It verifies the integrity and format of the index through signature, version, and checksum validations.

### Parameters

- None

### Returns

- `List[IndexEntry]` — a list of index entries, each containing metadata like timestamps, file mode, SHA-1 hash, flags, and file path.

### Operation Details

1. Attempts to read the binary contents of `.git/index`. If the file does not exist, returns an empty list.
2. Validates the SHA-1 checksum of the index file to ensure data integrity.
3. Parses the header to confirm the file signature (`DIRC`) and version number (expected to be 2).
4. Iteratively reads each index entry, unpacking fixed-length metadata and reading the variable-length file path.
5. Accounts for padding to an 8-byte boundary after each entry.
6. Returns a sorted list of `IndexEntry` objects.

### Usage Example

```python
entries = read_index()
for entry in entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

## `write_index(entries)`

### Purpose

Writes a list of `IndexEntry` objects back to the Git index file, updating the staging area with the current file metadata.

### Parameters

- `entries: List[IndexEntry]` — the list of index entries to write.

### Operation Details

1. Packs each `IndexEntry`'s metadata into a binary structure, including timestamps, device/inode numbers, mode, user/group IDs, file size, SHA-1 hash, and flags.
2. Encodes the file path and applies padding to align entries to an 8-byte boundary.
3. Constructs the index header with the signature, version, and number of entries.
4. Concatenates the header and all packed entries.
5. Computes the SHA-1 checksum of the entire index content and appends it to the file.
6. Writes the resulting data to `.git/index`.

### Usage Example

```python
from pygit.index import IndexEntry

# Assume 'entries' is a list of updated IndexEntry objects
write_index(entries)
print("Index updated with new entries.")
```

---

## `add(paths)`

### Purpose

Adds specified files to the Git index, staging them for the next commit. This function updates the index entries by hashing the file contents, gathering file metadata, and writing the new index to disk.

### Parameters

- `paths: List[str]` — list of file paths to add to the index.

### Operation Details

1. Normalizes file paths to use forward slashes.
2. Reads existing index entries and filters out any entries for the files being added (to avoid duplicates).
3. For each path:
   - Reads the file content and computes its SHA-1 blob hash.
   - Retrieves file system metadata (`stat`) including timestamps, mode, size, etc.
   - Creates a new `IndexEntry` object with the collected information.
4. Combines existing and new entries, sorts them by path.
5. Writes the updated list of entries back to the index file.

### Usage Example

```python
add(['src/main.py', 'README.md'])
print("Added files to index.")
```

---

# ASCII Diagram: Git Index File Structure

```
+------------------+
| Header           | 12 bytes
| - Signature      | 4 bytes ('DIRC')
| - Version        | 4 bytes (2)
| - Entry Count    | 4 bytes
+------------------+
| Index Entry 1    | variable length
| - Metadata       | 62 bytes fixed
| - Pathname       | variable (null-terminated)
| - Padding        | 0-7 bytes (to 8-byte alignment)
+------------------+
| Index Entry 2    | variable length
| ...              |
+------------------+
| SHA-1 Checksum   | 20 bytes (over entire file except these)
+------------------+
```

---

# Additional Notes

- The `IndexEntry` data structure typically includes:

  - `ctime_s`, `ctime_n`: file creation time seconds and nanoseconds
  - `mtime_s`, `mtime_n`: file modification time seconds and nanoseconds
  - `dev`, `ino`: device and inode numbers
  - `mode`: file mode (permissions and type)
  - `uid`, `gid`: user and group ID
  - `size`: file size in bytes
  - `sha1`: 20-byte SHA-1 hash of the file content (blob)
  - `flags`: various flags including path length
  - `path`: file path string

- The index is crucial for commit creation (`pygit.commit`), as seen in the integration with functions like `write_tree()` which reads the index entries to build a Git tree object.

---

# Related Functions and Flow

- `read_index()` is frequently used to inspect the current staging area.
- `write_index()` is called after modifying entries, such as after adding files.
- `add()` automates staging files by updating the index.
- These functions are foundational for higher-level commands like `commit`, `status`, and `diff`.

Together, they enable the pygit system to track changes, prepare commit objects, and maintain repository consistency.