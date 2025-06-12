# index.md

## Overview

This document provides detailed technical documentation for managing the Git index within the `pygit` project. It covers reading and writing the Git index file, adding files to the index (staging), and listing index entries. These operations are fundamental for staging changes before committing, ensuring the index state accurately reflects the user's intended snapshot of the working directory. Within the broader documentation tree, this file resides under the "Index and Working Copy Management" section, interfacing closely with repository commit operations (`pygit.commit`) and status checking (`pygit.status`). Understanding the index file handling is crucial for implementing core Git workflows such as `add`, `commit`, and `status`.

---

## Function Documentation

### `read_index()`

#### Purpose
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the staged files and their metadata.

#### Parameters
- None

#### Returns
- `List[IndexEntry]`: A list of index entries parsed from the index file.

#### Operation
1. Attempts to read the binary `.git/index` file.
2. Validates the SHA-1 checksum at the end of the file to ensure integrity.
3. Parses the index header, verifying the signature (`DIRC`) and version number (currently version 2).
4. Iterates over the entries in the index file:
   - Each entry consists of fixed-size fields followed by a variable-length path.
   - Corrects for padding to align entry sizes to 8-byte boundaries.
5. Decodes each entry into an `IndexEntry` object containing file metadata such as timestamps, device/inode numbers, mode, UID/GID, size, SHA-1 hash, flags, and the file path.

#### Example
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### `write_index(entries)`

#### Purpose
Writes a list of `IndexEntry` objects back to the Git index file, updating the staging area with new or modified entries.

#### Parameters
- `entries` (`List[IndexEntry]`): The list of index entries to write.

#### Operation
1. Packs each `IndexEntry` into a binary structure following Git's index file format.
2. Adds the necessary padding to align entries to 8-byte boundaries.
3. Constructs the index file header with signature, version, and number of entries.
4. Concatenates all packed entries and appends a SHA-1 checksum of the entire index data.
5. Writes the resulting binary data to `.git/index`.

#### Example
```python
write_index(entries)  # entries is a list of IndexEntry objects
```

---

### `add(paths)`

#### Purpose
Stages the specified file paths by adding them to the Git index. This function hashes the file contents and updates the index entries accordingly.

#### Parameters
- `paths` (`List[str]`): List of file paths to add to the index.

#### Operation
1. Normalizes path separators to '/'.
2. Reads the current index entries and filters out any entries matching the paths to be added.
3. For each path:
   - Reads the file contents and computes the SHA-1 hash of the blob object.
   - Retrieves file system metadata (timestamps, mode, UID, GID, size).
   - Creates a new `IndexEntry` with the collected data and calculated flags.
4. Appends the new entries to the filtered list.
5. Sorts all entries by path to maintain order.
6. Writes the updated entries back to the index file.

#### Example
```python
add(['src/main.py', 'README.md'])
```

---

### `ls_files(details=False)`

#### Purpose
Lists the files currently staged in the Git index.

#### Parameters
- `details` (`bool`, optional): If `True`, prints detailed info including file mode, SHA-1 hash, and stage number. Defaults to `False`.

#### Operation
1. Reads the current index entries.
2. For each entry:
   - If `details` is `False`, prints just the file path.
   - If `details` is `True`, prints mode (octal), SHA-1, stage number, and path.

#### Example
```python
ls_files()  # Prints just file paths
ls_files(details=True)  # Prints detailed info per file
```

---

### ASCII Diagram: Git Index File Layout

```
+-----------------------+
| Header (12 bytes)      |  -- Signature: "DIRC" (4 bytes)
|                        |  -- Version: 4 bytes (uint32)
|                        |  -- Number of entries: 4 bytes (uint32)
+-----------------------+
| Entry 1 (variable)     |  -- Fixed fields (62 bytes)
|                       |  -- Path (variable length)
|                       |  -- Padding to align to 8 bytes
+-----------------------+
| Entry 2 (variable)     |
+-----------------------+
| ...                   |
+-----------------------+
| SHA-1 checksum (20 B) |
+-----------------------+
```

Each index entry contains detailed metadata about a tracked file, including timestamps, device/inode numbers, mode, ownership, size, SHA-1 of the blob, flags, and the file path.

---

### Supporting Functions (Brief Overview)

While not directly part of this file, these functions interact closely with the index operations:

- **`hash_object(data, obj_type, write=True)`**: Hashes and optionally writes Git objects (blobs, trees, commits) into the object store.
- **`read_file(path)` / `write_file(path, data)`**: Utility functions to read/write raw bytes from/to files.
- **`read_index()` / `write_index(entries)`**: Core functions for index file manipulation.
- **`get_status()`**: Checks working directory against the index to determine changed, new, or deleted files.
- **`commit(message, author=None)`**: Uses the index snapshot to create a commit object.
- **`write_tree()`**: Creates a tree object from the current index entries.

---

## Summary

This documentation covers the essential functions for managing the Git index within `pygit`. The index acts as the staging area for files before committing and is implemented as a binary file with strict format requirements. The key functions allow reading the index into memory, modifying it by adding files, and writing back to disk. Additionally, listing index contents helps users inspect the staging area. Mastery of these functions is crucial for implementing Git’s fundamental staging and commit workflows.