# ls_files.md

# Listing Files in the Git Index with Optional Details

---

## Overview

The `ls_files.md` document details the functionality for listing files recorded in the Git index within the `pygit` project. This feature allows users to enumerate the files currently staged for commit, optionally displaying detailed metadata for each file such as file mode, SHA-1 hash, and stage number.

Situated under the **"Working Copy Status and Diff > Index Management"** section of the broader documentation, `ls_files.md` complements other index-related operations including adding files (`add.md`), reading (`read_index.md`), and writing (`write_index.md`) the Git index. It is an essential utility for inspecting the current state of the index before committing or pushing changes.

The primary function documented here is `ls_files`, which relies on reading the Git index entries and printing them in a user-friendly format. This functionality integrates with other commands such as `status` and `diff` to provide a comprehensive view of repository changes.

---

## Function Documentation

### `ls_files(details=False)`

**Purpose:**  
Prints the list of files currently staged in the Git index. When the optional `details` argument is set to `True`, it prints additional information for each file including the file mode (permissions), SHA-1 hash, and the stage number (useful for merge conflicts).

**Parameters:**

- `details` (`bool`, optional):  
  If `True`, prints detailed information per file. Defaults to `False`.

**Operation:**

1. Calls `read_index()` to obtain a list of `IndexEntry` objects representing files in the index.
2. Iterates over each index entry.
3. If `details` is `False`, prints only the file path.
4. If `details` is `True`, calculates the stage number by extracting bits 12 and 13 from the `flags` field of the index entry.
5. Prints the file mode (in octal), SHA-1 hash (hexadecimal), stage number, and file path in a formatted string.

**Example Usage:**

```python
import pygit

# List file paths only
pygit.ls_files()

# Output:
# README.md
# src/main.py
# tests/test_main.py

# List files with detailed info
pygit.ls_files(details=True)

# Output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	README.md
# 100755 7d9f5d4f5c7e7b4a7c3c8af19fdfd3abf8a4929f 0	src/main.py
# 100644 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c 0	tests/test_main.py
```

---

### Supporting Function: `read_index()`

**Purpose:**  
Reads the Git index file and returns a list of `IndexEntry` objects, each representing a file staged in the index.

**Highlights:**

- Validates the index file's signature and version.
- Parses each index entry including metadata fields and the file path.
- Checks the integrity of the index via SHA-1 checksum.

**Typical use:**  
Used internally by `ls_files()` to obtain the current index state.

---

## Additional Context and Related Commands

- **`status()`**: Displays changed, new, and deleted files compared to the index.  
- **`add(paths)`**: Adds files to the index.  
- **`diff()`**: Shows differences between the index and working directory files.

These commands along with `ls_files()` provide a comprehensive toolkit for managing and inspecting files staged for commit in the `pygit` repository.

---

## ASCII Diagram: Index Entry Structure

```
+-------------------+------------------+--------------+--------------------------+
| Metadata Fields   | SHA-1 Hash (20B) | Flags (2B)   | Path (variable length)   |
+-------------------+------------------+--------------+--------------------------+
| ctime_s (4 bytes) | ...              |              |                          |
| ctime_n (4 bytes) |                  |              |                          |
| mtime_s (4 bytes) |                  |              |                          |
| mtime_n (4 bytes) |                  |              |                          |
| dev (4 bytes)     |                  |              |                          |
| ino (4 bytes)     |                  |              |                          |
| mode (4 bytes)    |                  |              |                          |
| uid (4 bytes)     |                  |              |                          |
| gid (4 bytes)     |                  |              |                          |
| size (4 bytes)    |                  |              |                          |
+-------------------+------------------+--------------+--------------------------+

* The 'Flags' field includes the stage number in its upper bits.
```

---

## Summary

The `ls_files` function is a straightforward yet powerful tool for inspecting the contents of the Git index in `pygit`. By optionally showing detailed metadata, it assists developers in verifying the exact state of files prepared for commit, facilitating better version control workflows.

For more detailed file operations and repository management, see adjacent documentation files such as `add.md` and `status.md`.