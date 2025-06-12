# write_index.md

# Writing the Git Index File (`write_index`)

---

## Overview

This document details the process of writing the Git index file, focusing on the `write_index` function. The Git index (also known as the staging area) is a binary file that records the state of the working directory as prepared for the next commit. Writing this file correctly ensures that changes are staged and tracked accurately, playing a critical role in Git’s internal workflow.

Within the broader documentation tree, this file falls under **Index and Working Copy Management**, complementing related files such as `read_index.md` (for reading the index) and `add.md` (for adding files to the index). Together, these documents provide comprehensive coverage on managing the Git index, crucial for functions like `commit`, `diff`, and `status`.

---

## Function Documentation

### `write_index(entries)`

**Purpose:**  
Write a list of `IndexEntry` objects to the Git index file (`.git/index`). This function serializes the in-memory index entries into the precise binary format expected by Git, including a checksum for integrity verification.

**Parameters:**  
- `entries` (`List[IndexEntry]`): A list of `IndexEntry` instances representing the staged files and their metadata.

**Preconditions:**  
- Each `IndexEntry` must be correctly populated with file metadata such as timestamps, device and inode info, file mode, ownership, size, SHA-1 of the object, flags, and the file path.
- The entries list should be sorted by path to maintain consistency and correctness.

---

### How `write_index` Works - Step-by-Step

1. **Pack Each Index Entry:**  
   For each `IndexEntry`, pack the fixed metadata fields using the struct format `!LLLLLLLLLL20sH`, which includes:
   - Creation time (seconds and nanoseconds)
   - Modification time (seconds and nanoseconds)
   - Device number
   - Inode number
   - File mode
   - User ID
   - Group ID
   - File size
   - SHA-1 hash (20 bytes)
   - Flags (2 bytes)

2. **Encode File Path and Align Entry Length:**  
   - The file path is encoded as UTF-8 bytes and appended to the packed fixed fields.
   - The total length of each entry (fixed fields + path + null terminator + padding) is aligned to an 8-byte boundary by adding null bytes (`b'\x00'`).

3. **Create the Index Header:**  
   - The header consists of:
     - Signature: ASCII bytes `"DIRC"`
     - Version: 2 (Git index version)
     - Number of entries

4. **Concatenate Header and Entries:**  
   - The header and all packed entries are concatenated into one binary blob.

5. **Calculate and Append SHA-1 Checksum:**  
   - Compute the SHA-1 hash of all data except the checksum itself.
   - Append this 20-byte checksum to the end of the data.

6. **Write to Disk:**  
   - Write the complete data (header + entries + checksum) to the index file path `.git/index`.

---

### Example Usage

```python
# Assume we have a list of IndexEntry objects named `entries`
from pygit import write_index

# Write the index entries to the .git/index file
write_index(entries)
print("Git index file written successfully.")
```

---

## Additional Context: Structure of an Index Entry

Below is an ASCII diagram illustrating the layout of a single index entry in the Git index file:

```
+--------------------+--------------------+--------------+-------------------+
| Fixed Fields (62 B) | Path (variable len) | Null Byte (1) | Padding (0-7 bytes)|
+--------------------+--------------------+--------------+-------------------+

Fixed Fields (packed struct):
  - ctime seconds (4 bytes)
  - ctime nanoseconds (4 bytes)
  - mtime seconds (4 bytes)
  - mtime nanoseconds (4 bytes)
  - device (4 bytes)
  - inode (4 bytes)
  - mode (4 bytes)
  - uid (4 bytes)
  - gid (4 bytes)
  - size (4 bytes)
  - SHA-1 hash (20 bytes)
  - Flags (2 bytes)
```

**Note:** The entire entry size is aligned to an 8-byte boundary by adding null bytes after the path and null terminator.

---

## Related Functions and Workflow Integration

- `read_index()`: Reads and parses the index file into `IndexEntry` objects for manipulation.
- `add(paths)`: Adds files to the index by creating/updating `IndexEntry` objects and calling `write_index`.
- `hash_object(data, obj_type)`: Hashes file data to produce SHA-1 hashes stored in index entries.
- `write_file(path, data)`: Low-level utility for writing binary data to files, used by `write_index`.
- `commit()`: Uses the index file (written by `write_index`) to create tree and commit objects.

---

## Summary

The `write_index` function is essential for persisting the in-memory representation of staged files to disk in the exact binary format Git expects. This process guarantees that subsequent Git operations like commit or diff work on a consistent snapshot of the working directory's staged state.

By precisely packing metadata, paths, and checksums, `write_index` ensures the integrity and correctness of the Git index, making it a cornerstone of Git’s internal state management.

---

_End of `write_index.md` documentation._