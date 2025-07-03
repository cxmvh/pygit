# Git Index File Documentation

## Overview

This document provides detailed technical reference for managing the Git index file within the repository. It covers the reading and writing of the Git index file (`.git/index`), mechanisms to add files to the index, and explains the `IndexEntry` data structure along with the binary format of the index file. This file is part of the **Index Management** section of the repository documentation, which focuses on operations manipulating the Git index and its entries.

The Git index acts as a staging area between the working directory and the repository history. Understanding and correctly handling the index file is crucial for implementing commands like `git add`, `git commit`, and for tracking changes efficiently.

---

## IndexEntry Structure and Index File Format

The index file consists of a binary header followed by a sequence of index entries and ends with a SHA-1 checksum for integrity verification.

### Index File Layout

```
+----------------+-----------------+---------------------+---------+
| Header (12 B)  | Entries (variable) | SHA-1 Checksum (20 B) |
+----------------+-----------------+---------------------+---------+

Header structure (12 bytes):
- Signature: 4 bytes, ASCII "DIRC"
- Version: 4 bytes, unsigned int (currently 2)
- Number of entries: 4 bytes, unsigned int

Each Entry structure (variable length):
- Fixed fields: 62 bytes (timestamps, device, inode, mode, uid, gid, size, SHA-1, flags)
- Path: variable length, null-terminated string
- Padding: variable bytes, to align entry size to multiple of 8 bytes
```

The checksum at the end validates the contents of the index (except the checksum itself).

### IndexEntry Data Structure

Each entry corresponds to a file staged in the index and contains metadata necessary for Git operations.

| Field         | Description                          | Size (bytes) |
|---------------|------------------------------------|--------------|
| ctime_s       | Creation time (seconds)             | 4            |
| ctime_n       | Creation time (nanoseconds)         | 4            |
| mtime_s       | Modification time (seconds)         | 4            |
| mtime_n       | Modification time (nanoseconds)     | 4            |
| dev           | Device number                      | 4            |
| ino           | Inode number                      | 4            |
| mode          | File mode (permissions)            | 4            |
| uid           | User ID of owner                  | 4            |
| gid           | Group ID of owner                 | 4            |
| size          | File size                        | 4            |
| sha1          | SHA-1 hash of file content        | 20           |
| flags         | Flags including path length       | 2            |
| path          | File path, null-terminated string | variable     |

---

## Functions

### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects.

**Purpose:**

- Parse the binary `.git/index` file.
- Verify checksum and header validity.
- Extract all entries into a usable Python data structure.

**Parameters:** None

**Returns:**

- List of `IndexEntry` objects representing staged files.

**Operation Details:**

1. Reads the entire index file contents.
2. Verifies the SHA-1 checksum matches the contents.
3. Parses header fields: signature, version, number of entries.
4. Iterates over the entries segment, unpacking fixed fields using `struct.unpack`.
5. Reads the path string for each entry.
6. Accounts for padding to align entries to 8-byte boundaries.
7. Collects all entries and verifies the count matches the header.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### `write_index(entries)`

Writes a list of `IndexEntry` objects back to the `.git/index` file.

**Purpose:**

- Serialize the list of index entries into the Git index file format.
- Calculate and append the SHA-1 checksum for integrity.

**Parameters:**

- `entries`: List of `IndexEntry` objects to write.

**Returns:** None

**Operation Details:**

1. Packs each entry's fixed fields with `struct.pack`.
2. Encodes the path and adds null-termination.
3. Pads each entry to an 8-byte boundary.
4. Concatenates all packed entries.
5. Builds the header with signature, version, and number of entries.
6. Computes the SHA-1 checksum of all data except the checksum itself.
7. Writes the concatenated data plus checksum to `.git/index`.

**Example Usage:**

```python
from collections import namedtuple

IndexEntry = namedtuple('IndexEntry', [
    'ctime_s', 'ctime_n', 'mtime_s', 'mtime_n', 'dev', 'ino', 'mode',
    'uid', 'gid', 'size', 'sha1', 'flags', 'path'
])

entries = read_index()

# Modify entries or add new ones
# ...

write_index(entries)
```

---

### `add(paths)`

Add one or multiple file paths to the Git index.

**Purpose:**

- Hash contents of given files and add/update their entries in the index.
- Update metadata in the index for the added files.

**Parameters:**

- `paths`: List of file paths (strings) to add to the index.

**Returns:** None

**Operation Details:**

1. Normalize paths to use forward slashes.
2. Read current index entries, excluding entries for paths being added.
3. For each path:
   - Read file content.
   - Compute its SHA-1 hash and store the object.
   - Retrieve file system metadata.
   - Create a new `IndexEntry` with updated metadata and content hash.
4. Append new entries to the list.
5. Sort entries by path.
6. Write updated entries back to the index file.

**Example Usage:**

```python
add(['README.md', 'src/main.py'])
```

---

### `ls_files(details=False)`

List files currently stored in the Git index.

**Purpose:**

- Display the list of files staged in the index.
- Optionally includes detailed metadata such as mode, SHA-1 hash, and stage number.

**Parameters:**

- `details` (bool): If `True`, print detailed info; otherwise, just file paths.

**Returns:** None

**Operation Details:**

1. Calls `read_index()` to get current entries.
2. If `details` is `False`, prints file paths line by line.
3. If `details` is `True`, prints mode (octal), SHA-1 hash, stage, and path.

**Example Usage:**

```python
ls_files()          # Prints filenames only
ls_files(details=True)  # Prints detailed info for each index entry
```

Output example with details:

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	README.md
100755 3a4e3f7f2b1d6e2d0f9c8a1d3b4e5f6a7b8c9d0e 0	src/main.py
```

---

### `read_file(path)`

Read the contents of a file from the filesystem.

**Purpose:**

- Read raw bytes from a file at the specified path.

**Parameters:**

- `path` (string): Path to the file.

**Returns:**

- Bytes of file contents.

**Example Usage:**

```python
content = read_file('README.md')
print(content.decode())
```

---

### `write_file(path, data)`

Write raw bytes to a file on the filesystem.

**Purpose:**

- Write bytes data to the file at the given path.

**Parameters:**

- `path` (string): Path to the file.
- `data` (bytes): Data to write.

**Returns:** None

**Example Usage:**

```python
write_file('.git/index', index_data)
```

---

## ASCII Diagram: Index File Structure

```
+-------------------------+
|         Header          |
| +---------------------+ |
| | Signature: "DIRC"   | | 4 bytes
| +---------------------+ |
| | Version: 2         | | 4 bytes (uint32)
| +---------------------+ |
| | Entry count         | | 4 bytes (uint32)
| +---------------------+ |
+-------------------------+

+-------------------------+
|      Index Entries      |  (variable length)
| +---------------------+ |
| | ctime_s             | | 4 bytes
| | ctime_n             | | 4 bytes
| | mtime_s             | | 4 bytes
| | mtime_n             | | 4 bytes
| | dev                 | | 4 bytes
| | ino                 | | 4 bytes
| | mode                | | 4 bytes
| | uid                 | | 4 bytes
| | gid                 | | 4 bytes
| | size                | | 4 bytes
| | sha1                | | 20 bytes
| | flags               | | 2 bytes
| +---------------------+ |
| | Path (null-terminated) |
| +---------------------+ |
| | Padding (to align 8 bytes) |
+-------------------------+

+-------------------------+
|   SHA-1 Checksum (20B)  |
+-------------------------+
```

---

## Notes on the Index Flags Field

- The `flags` field is 2 bytes.
- The lower 12 bits represent the length of the path (if less than 0xFFF).
- The upper 4 bits include stage information and other flags.

---

## Summary

This documentation covers the core operations to:

- Read the Git index file and parse its entries.
- Write entries back to the index file with proper formatting and checksum.
- Add files to the index, including hashing and metadata extraction.
- List files staged in the index with optional detailed information.

Understanding these functions and the index file format is essential for implementing Git features that interact with the staging area, such as `git add`, `git status`, and `git commit`.

For related operations on objects, commits, and repository manipulation, see the other sections in the documentation tree like [Object Handling](objects.md) and [Commit Management](commit.md).