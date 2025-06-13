# ls_files.md

# Documentation for Listing Files in the Git Index with Optional Details

---

## Overview

The `ls_files.md` file documents the functionality related to listing files tracked in the Git index within the `pygit` project. This feature is part of the **Status and Working Copy Management** section of the documentation, which focuses on commands and internals related to checking the working copy status, listing files, and managing the Git index.

Specifically, this document centers on the `ls_files` function and its supporting utilities, which allow users or other components to view the current state of files stored in the Git index. This includes optional detailed output of file metadata such as file mode, SHA-1 hash, and staging information.

The file fits into the broader repository structure as a key utility for inspecting the index, complementing other status and diff features, and providing foundational support for commands that operate on tracked files.

---

## Functions

### 1. `ls_files(details=False)`

#### Purpose

Lists all files currently recorded in the Git index. When run with the default argument `details=False`, it prints only the paths of tracked files. If `details=True`, it prints additional information for each file entry: the file mode, SHA-1 hash, and stage number within the index.

#### Parameters

- `details` (bool): Optional flag to include detailed metadata in the output. Defaults to `False`.

#### Operation

- Reads the Git index entries by calling `read_index()`.
- Iterates over each entry:
  - If `details` is `False`, prints the file path.
  - If `details` is `True`, extracts the stage number from the `flags` field, then prints the mode (in octal), SHA-1 hash (hexadecimal), stage number, and path in a formatted string.

#### Usage Example

```python
# List just the paths of files in the index
ls_files()

# List detailed info (mode, sha1, stage, path) for each file in the index
ls_files(details=True)
```

#### Output Example with `details=True`

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 7f8a8e0d9d8f4c4a6a7b3a0c8a1dd3c4f2e3d5bc 0    src/main.py
100755 c1b9e9e8f7a9d7c8e6b5a4d3c2b1a0987f6e5d4c 0    scripts/deploy.sh
```

---

### 2. `read_index()`

#### Purpose

Reads the Git index file (`.git/index`) and parses it into a list of `IndexEntry` objects representing the current tracked files and their metadata.

#### Operation

- Attempts to read the raw content of the `.git/index` file. If the file is missing, returns an empty list.
- Verifies the checksum of the index file by comparing the stored SHA-1 digest with a computed digest of the file content (excluding the last 20 bytes).
- Parses the file header for signature, version, and number of entries.
- Iteratively parses each index entry:
  - Each entry consists of fixed-size metadata fields followed by a null-terminated path string.
  - Entries are aligned to an 8-byte boundary.
- Returns a list of `IndexEntry` objects.

#### Usage Example

```python
entries = read_index()
for entry in entries:
    print(f'{entry.path}: {entry.sha1.hex()}')
```

---

### 3. `read_file(path)`

#### Purpose

Reads the raw bytes content of a file from the file system.

#### Operation

- Opens the file in binary mode.
- Reads and returns all bytes.

#### Usage Example

```python
content = read_file('README.md')
print(content.decode())
```

---

## Supporting Concepts

### Git Index Entry Structure

Each entry in the Git index contains metadata about a tracked file, such as:

- Creation and modification times
- Device and inode numbers
- File mode (permissions and type)
- User and group IDs
- File size
- SHA-1 hash of the file content
- Flags (including stage number)
- File path relative to the repository root

### Extracting Stage Number from Flags

The stage number is stored in bits 12 and 13 of the `flags` field in an index entry. It can be extracted as follows:

```python
stage = (entry.flags >> 12) & 3
```

---

## ASCII Diagram: Git Index Entry Listing Flow

```
+---------------------+
| Start ls_files()     |
+----------+----------+
           |
           v
+---------------------+
| read_index()        | -- reads and parses -->
+----------+----------+     .git/index file
           |
           v
+---------------------+
| For each entry in   |
| index entries       |
+----------+----------+
           |
           +------------------------------+
           |                              |
           v                              v
+---------------------+          +---------------------+
| details == False?    |  No      | details == True?     | Yes
+----------+----------+          +----------+----------+
           |                              |
           v                              v
+---------------------+          +-----------------------------+
| print entry.path     |          | Extract stage from flags     |
+---------------------+          | print mode, sha1, stage, path|
                                 +-----------------------------+
```

---

## Summary

The `ls_files.md` documentation covers the core functionality for listing files in the Git index. The main interface is the `ls_files` function, which supports plain or detailed output of tracked files. It relies on the `read_index` function to parse the Git index file accurately and `read_file` to fetch file contents when needed elsewhere. This capability is essential for status reporting, commit preparation, and other Git operations that require knowledge of the tracked file set and their metadata.