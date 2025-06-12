# index_io.md

# Reading and Writing the Git Index File (`read_index` and `write_index`)

---

## Overview

This document details the implementation and usage of functions responsible for reading and writing the Git index file within the `pygit` project. The Git index (also known as the staging area) is a critical structure that tracks the state of files to be committed. This file resides at `.git/index` and stores metadata about the files staged for the next commit, including their paths, modes, modification timestamps, and content hashes.

Within the broader `Index and Working Copy Management` section of the documentation tree, this file complements other index-related topics by focusing specifically on the input/output operations for the index file. These operations are fundamental for commands like `git add`, `git ls-files`, and `git commit`, which rely on accurately reading from and writing to the index.

The key functions covered here are:

- `read_index()`: Parses the binary index file into structured entries.
- `write_index(entries)`: Serializes a list of index entries back into the binary index file format.

---

## Function Documentation

---

### `read_index()`

**Purpose:**  
Reads the Git index file (`.git/index`), parses its contents, verifies integrity via checksum, and returns a list of `IndexEntry` objects representing each staged file.

**Signature:**  
```python
def read_index() -> List[IndexEntry]
```

**Parameters:**  
- None

**Returns:**  
- `List[IndexEntry]`: A list of parsed index entries, each containing metadata about one file.

**Preconditions:**  
- The `.git/index` file exists and is readable.
- The index file complies with version 2 specification of Git index format.

**Operation Details:**

1. Attempts to read the raw bytes from `.git/index`. If the file is missing, returns an empty list.
2. Validates the index file signature (`DIRC`) and version number (expecting version 2).
3. Extracts the total number of entries.
4. Verifies the SHA-1 checksum trailing the file for integrity.
5. Iterates over the entries sequentially:
   - Each entry consists of fixed-length fields (62 bytes) followed by a null-terminated path string.
   - Parses metadata fields including timestamps, device/inode numbers, file mode, user/group IDs, file size, SHA-1 hash of file content, and flags.
   - Calculates the padded length of the entry to align to an 8-byte boundary, then advances the parsing cursor accordingly.
6. Decodes the path from bytes to a string and constructs an `IndexEntry` object.
7. Returns the full list of index entries.

**Example Usage:**

```python
from pygit import read_index

index_entries = read_index()
for entry in index_entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

**Sample Output:**

```
File: README.md, SHA-1: a1b2c3d4e5f67890123456789abcdef012345678, Mode: 0o100644
File: src/main.py, SHA-1: b2c3d4e5f67890123456789abcdef0123456789a, Mode: 0o100755
```

---

### `write_index(entries)`

**Purpose:**  
Serializes a list of `IndexEntry` objects and writes them to the Git index file at `.git/index` in the correct binary format, including a SHA-1 checksum for file integrity.

**Signature:**  
```python
def write_index(entries: List[IndexEntry]) -> None
```

**Parameters:**  
- `entries` (`List[IndexEntry]`): The list of index entries to write to the index file.

**Preconditions:**  
- Each `IndexEntry` should conform to the Git index specification.
- Entries are typically sorted by file path for consistency.

**Operation Details:**

1. Iterates over each `IndexEntry` in the provided list:
   - Packs the fixed-length fields (timestamps, device/inode, mode, owner IDs, size, SHA-1 hash, flags) into a binary structure using `struct.pack`.
   - Encodes the file path as bytes, appends a null terminator, and pads the entire entry to an 8-byte boundary.
   - Collects all packed entries into a list.
2. Creates the header for the index file:
   - Signature: `b'DIRC'`
   - Version: `2`
   - Number of entries
3. Concatenates the header and all packed entries.
4. Computes the SHA-1 checksum over the entire data (excluding the checksum itself).
5. Writes the concatenated data plus the checksum to `.git/index`.

**Example Usage:**

```python
from pygit import read_index, write_index, IndexEntry

# Read existing entries
entries = read_index()

# Modify entries, for example, add a new file entry
new_entry = IndexEntry(
    ctime_s=1610000000, ctime_n=0,
    mtime_s=1610000000, mtime_n=0,
    dev=2050, ino=305419896,
    mode=0o100644, uid=1000, gid=1000,
    size=1234,
    sha1=bytes.fromhex("a1b2c3d4e5f67890123456789abcdef012345678"),
    flags=len("newfile.txt"),
    path="newfile.txt"
)
entries.append(new_entry)

# Sort entries by path before writing
entries.sort(key=lambda e: e.path)

# Write back to index file
write_index(entries)
```

---

## Supporting Concepts and Data Structures

### `IndexEntry` Structure

Each index entry corresponds to a file staged in Git and contains the following fields:

| Field      | Description                                        | Size (bytes) |
|------------|--------------------------------------------------|--------------|
| `ctime_s`  | Creation time, seconds                            | 4            |
| `ctime_n`  | Creation time, nanoseconds                        | 4            |
| `mtime_s`  | Modification time, seconds                        | 4            |
| `mtime_n`  | Modification time, nanoseconds                    | 4            |
| `dev`      | Device number                                     | 4            |
| `ino`      | Inode number                                     | 4            |
| `mode`     | File mode (permissions and type)                  | 4            |
| `uid`      | User ID                                          | 4            |
| `gid`      | Group ID                                         | 4            |
| `size`     | File size in bytes                                | 4            |
| `sha1`     | SHA-1 hash of file content (20 bytes)            | 20           |
| `flags`    | Flags including length of path (lower 12 bits)   | 2            |
| `path`     | File path (variable length, null-terminated)     | variable     |

---

## ASCII Diagram: Git Index File Structure

```
+-----------------+-----------------------+---------------------------+------------------+
| Header (12 bytes)| Entries (variable)     | SHA-1 Checksum (20 bytes) | End of File      |
+-----------------+-----------------------+---------------------------+------------------+

Header:
+----------------+------------+----------------+
| Signature (4B) | Version(4B)| Num Entries(4B)|
+----------------+------------+----------------+

Entry (per file):
+--------------------------------------------------------------+
| ctime_s (4B) | ctime_n (4B) | mtime_s (4B) | mtime_n (4B)   |
| dev (4B) | ino (4B) | mode (4B) | uid (4B) | gid (4B) | size (4B)|
| sha1 (20B) | flags (2B) | path (variable + null) + padding to 8-byte align |
+--------------------------------------------------------------+
```

---

## Summary

The `read_index` and `write_index` functions provide the essential mechanisms for interacting with the Git index file, enabling the reading, parsing, modification, and serialization of staging area data. Proper handling of this file ensures correct tracking of staged files, and these functions underpin core Git operations implemented in the `pygit` project.

---

# Appendix: Related Functions

- `read_file(path)`: Reads file contents as bytes (used internally by `read_index`).
- `write_file(path, data)`: Writes bytes to a file (used internally by `write_index`).
- `hash_object(data, obj_type)`: Computes SHA-1 hash for Git objects (used in index entry creation).
- `ls_files(details=False)`: Lists files in the index, demonstrating usage of `read_index`.
- `add(paths)`: Adds files to the index, reading and writing the index.

---

# End of index_io.md