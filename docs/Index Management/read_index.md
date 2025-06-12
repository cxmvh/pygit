# read_index.md

# Documentation for Reading the Git Index File

---

## Overview

The `read_index.md` document provides a detailed explanation of how the Git index file (also known as the staging area) is read and parsed within the project. The index file serves as a critical intermediary data structure that tracks the state of the working directory and what will be included in the next commit. This file is part of the broader **Index Management** section of the documentation, which encompasses reading, writing, and manipulating the Git index. Understanding how the index is read is essential for commands that list files (`ls_files`), display status (`status`), create commits, or generate diffs, as these operations rely on accurate interpretation of the index contents.

---

## Function Documentation

### `read_index()`

#### Purpose
`read_index()` reads the Git index file from the `.git/index` path and returns a list of parsed `IndexEntry` objects representing the files staged in the index. Each entry contains metadata such as file timestamps, device and inode numbers, mode, ownership, size, SHA-1 hash of the blob content, flags, and the file path.

This function is foundational for any operation that inspects the current staging area state, including listing files, checking file statuses, and preparing commits.

#### Parameters
- None

#### Returns
- `List[IndexEntry]`: A list of `IndexEntry` objects parsed from the index file. If the index file does not exist, an empty list is returned.

#### Preconditions
- Assumes the `.git/index` file exists and is a valid Git index file version 2.
- The index file is properly checksummed (SHA-1 hash at the end of the file matches the contents).

#### Operation Details

1. **Read Index File**: Attempts to read the entire `.git/index` file into memory as bytes.
2. **Checksum Verification**: The last 20 bytes of the file are the SHA-1 checksum. Compute the SHA-1 hash of the file's contents excluding these last 20 bytes and verify it matches the stored checksum. If not, raise an assertion error.
3. **Header Parsing**: Unpack the first 12 bytes as:
   - Signature (4 bytes): Must be `b'DIRC'`.
   - Version (4 bytes): Must be 2.
   - Number of entries (4 bytes): Number of index entries in the file.
4. **Entry Parsing Loop**:
   - Each index entry is parsed sequentially from the remainder of the file.
   - Each entry starts with 62 bytes containing fixed-size fields, unpacked to:
     - `ctime_s`, `ctime_n`: Creation time seconds and nanoseconds.
     - `mtime_s`, `mtime_n`: Modification time seconds and nanoseconds.
     - `dev`, `ino`: Device and inode numbers.
     - `mode`: File mode.
     - `uid`, `gid`: User and group IDs.
     - `size`: File size.
     - `sha1`: 20-byte SHA-1 hash of the file content.
     - `flags`: Entry flags.
   - After these fields, a null-terminated path string follows.
   - Padding bytes (0s) align the total entry length to an 8-byte boundary.
5. **Entry Object Creation**: Each parsed entry is instantiated as an `IndexEntry` object and appended to the list.
6. **Validation**: Confirm the number of entries parsed matches the expected count from the header.
7. **Return**: Return the list of all parsed entries.

#### Example Usage

```python
from pygit import read_index

# Read the current git index entries
entries = read_index()

# Print paths of all staged files
for entry in entries:
    print(f"{entry.path} ({entry.mode:o}), SHA-1: {entry.sha1.hex()}")
```

---

## Supporting Concepts and ASCII Diagram

The Git index file format consists of a header, multiple index entries, and a trailing checksum:

```
+-------------------+----------------------------+-------------------+
| Header (12 bytes)  | Index Entries (variable)   | SHA-1 Checksum(20)|
+-------------------+----------------------------+-------------------+
| Signature (4)     | Entry 1                    |                   |
| Version (4)       | Entry 2                    |                   |
| Num Entries (4)   | ...                        |                   |
+-------------------+----------------------------+-------------------+

Each Index Entry:
+-------------------------+--------------+----------------+
| Fixed fields (62 bytes) | Path (N bytes)| Padding (0-7)  |
+-------------------------+--------------+----------------+

Where:
- Fixed fields include timestamps, device/inode, mode, uid/gid, size, sha1, flags.
- Path is a null-terminated string.
- Padding aligns entry size to multiple of 8 bytes.
```

---

## Related Functions in the Documentation Tree

- `ls_files(details=False)`: Utilizes `read_index()` to list files in the index.
- `write_index(entries)`: Writes `IndexEntry` objects back to the `.git/index` file.
- `get_status()`: Uses `read_index()` to compare index entries with the working directory.
- `add(paths)`: Reads the index, updates entries, and writes back.
- `write_tree()`: Reads the index entries to write a tree object.

---

## Summary

The `read_index()` function is a core utility for accessing the Git staging area’s current state. By parsing the `.git/index` file format strictly according to Git's specification, it enables the rest of the system to perform file listing, status checks, and commit preparation accurately and efficiently. Understanding this function is key for contributors working with index-related features or implementing Git-compatible tooling.