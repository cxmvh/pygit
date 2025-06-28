# index_handling.md

# Overview

This document details the handling of the Git index within the `pygit` project, focusing on reading and writing the index file and adding files to the index. The Git index acts as a staging area for files before they are committed. Correct management of the index is critical for Git operations such as committing changes and inspecting the working copy status. This file fits within the broader "Index and Working Copy Management" section of the documentation tree, bridging low-level index file manipulations with higher-level commands like commit and status.

The file covers key functions for interpreting the binary index format (`read_index`), persisting index entries back to disk (`write_index`), and updating the index with new or modified files (`add`). These functions interface with other parts of the system such as object hashing, file IO, and commit creation, making this document essential for developers working on staging and commit workflows.

---

# Function Documentation

---

## `read_index()`

### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the files currently staged. This function parses the binary format of the index including headers, entries, and verifies the file checksum to ensure integrity.

### Parameters

None.

### Returns

- `List[IndexEntry]`: List of index entries parsed from the index file. Each `IndexEntry` includes metadata such as file timestamps, device/inode info, mode, SHA-1 hash of the blob, flags, and file path.

### Operation Details

1. Attempts to read `.git/index`. If the file does not exist, returns an empty list.
2. Verifies the SHA-1 checksum at the end of the file matches the hash of the preceding bytes.
3. Parses the header (signature `DIRC`, version 2, number of entries).
4. Iterates through the entries, each with a fixed-size metadata section followed by a null-terminated path string, padded to an 8-byte boundary.
5. For each entry, populates an `IndexEntry` object.
6. Returns the list of all entries.

### Example Usage

```python
entries = read_index()
for entry in entries:
    print(f"{entry.mode:o} {entry.sha1.hex()} {entry.path}")
```

---

## `write_index(entries)`

### Purpose

Writes a list of `IndexEntry` objects back to the Git index file in the proper binary format, including headers and checksum.

### Parameters

- `entries (List[IndexEntry])`: List of index entries to write.

### Operation Details

1. Packs each `IndexEntry` into binary format:
   - Fixed metadata fields packed with `struct`.
   - Path encoded as bytes, padded to 8-byte alignment with null bytes.
2. Concatenates all packed entries.
3. Prepends a header with signature `DIRC`, version 2, and entry count.
4. Computes SHA-1 checksum of the entire content (excluding checksum).
5. Writes the content plus checksum to `.git/index`.

### Example Usage

```python
write_index(entries)
print("Index updated with", len(entries), "entries.")
```

---

## `add(paths)`

### Purpose

Adds one or more files to the Git index, preparing them to be committed. This includes hashing file contents, creating index entries, and updating the index file.

### Parameters

- `paths (List[str])`: List of file paths to add to the index. Paths should be relative to the repository root.

### Operation Details

1. Normalizes Windows-style backslashes to forward slashes.
2. Reads the current index entries, excluding any entries for the specified paths (to replace them).
3. For each path in `paths`:
   - Reads file content and creates a blob object, obtaining its SHA-1.
   - Retrieves file stats for timestamps, permissions, device, inode, user/group IDs, and size.
   - Constructs an `IndexEntry` with the file metadata, SHA-1, and path.
4. Appends new entries to the list and sorts all entries by path.
5. Writes the updated entries back to the index file.

### Preconditions

- Files must exist and be readable.
- Paths should be relative and normalized.

### Example Usage

```python
files_to_add = ['README.md', 'src/main.py']
add(files_to_add)
print("Added files to index:", files_to_add)
```

---

# ASCII Diagram: Git Index File Structure

```
+--------------------------------------------------------+
| Header                                                 |
|  - Signature: 4 bytes ('DIRC')                         |
|  - Version: 4 bytes (uint32, e.g. 2)                   |
|  - Entry count: 4 bytes (uint32)                        |
+--------------------------------------------------------+
| Entry 1                                                |
|  - ctime seconds (4 bytes)                             |
|  - ctime nanoseconds (4 bytes)                         |
|  - mtime seconds (4 bytes)                             |
|  - mtime nanoseconds (4 bytes)                         |
|  - dev (4 bytes)                                       |
|  - ino (4 bytes)                                       |
|  - mode (4 bytes)                                      |
|  - uid (4 bytes)                                       |
|  - gid (4 bytes)                                       |
|  - size (4 bytes)                                      |
|  - SHA-1 hash (20 bytes)                               |
|  - flags (2 bytes)                                     |
|  - path (variable length, null-terminated)            |
|  - padding (to align entry size to multiple of 8 bytes)|
+--------------------------------------------------------+
| Entry 2                                                |
|  ...                                                   |
+--------------------------------------------------------+
| ...                                                    |
+--------------------------------------------------------+
| SHA-1 checksum of all previous bytes (20 bytes)       |
+--------------------------------------------------------+
```

---

# Additional Notes

- The `IndexEntry` structure used in these functions is assumed to be defined elsewhere in the codebase, encapsulating file metadata and SHA-1 blob references.
- These functions rely on other utility functions like `read_file`, `write_file`, and `hash_object` to handle file IO and object hashing.
- Index integrity is enforced by verifying the checksum during reading and ensuring proper formatting and padding when writing.
- The `add` function integrates with object hashing to create blob objects for newly added files, linking the index to the object database.
- Changes in the index directly affect commits, as the index contents determine the tree object written during a commit.

---

# Summary

This documentation describes the core mechanisms for managing the Git index in `pygit`. Understanding and utilizing `read_index`, `write_index`, and `add` is essential for staging files and preparing the repository state for commits. Mastery of these functions provides a foundation for extending Git functionality related to the working copy and version history management.