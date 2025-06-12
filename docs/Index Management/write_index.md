# write_index.md

# Writing Entries to the Git Index File

## Overview

This document details the process and functions involved in writing entries to the Git index file (`.git/index`). The Git index serves as a staging area between the working directory and the repository, tracking metadata and blob object hashes of files to be committed. Writing to the index file involves serializing a collection of index entries, each representing a file, into a binary format with an overall checksum for integrity verification.

Within the broader project documentation, this file fits under the **Index Management** section, complementing `read_index.md` by focusing on persisting index entries back to disk. Understanding how the index is written is essential for implementing staging operations, preparing commits, and ensuring consistency between the working copy and Git's internal state.

---

## Function Documentation

### `write_index(entries)`

**Purpose:**  
Serialize a list of `IndexEntry` objects and write them to the `.git/index` file in the correct Git index file format.

**Parameters:**  
- `entries` (List[IndexEntry]): A list of index entries to write. Each entry contains file metadata such as timestamps, device numbers, inode, mode, user/group IDs, file size, SHA-1 hash, flags, and file path.

**Preconditions:**  
- The `entries` list should be sorted by the file path (lexicographically) before calling this function to maintain Git index ordering.
- Each `IndexEntry` must be correctly populated with valid data.

**Operation Details:**

1. **Entry Serialization:**  
   Each `IndexEntry` is serialized into a fixed-size header followed by the file path string, then padded with null bytes (`\x00`) to ensure that the total length of the entry is a multiple of 8 bytes. The fixed header is 62 bytes long and includes fields such as creation and modification times, device and inode numbers, file mode, user and group IDs, file size, SHA-1 hash, and flags.

2. **Index File Header:**  
   The index file begins with a 12-byte header:
   - 4-byte signature: ASCII string `"DIRC"`.
   - 4-byte version number (big-endian unsigned integer), currently `2`.
   - 4-byte entry count (big-endian unsigned integer).

3. **Concatenation:**  
   The serialized entries are concatenated following the header.

4. **Checksum:**  
   A SHA-1 hash is computed over all data except the final 20 bytes. This 20-byte SHA-1 digest is appended at the end of the index file for integrity verification.

5. **File Writing:**  
   The complete binary data (header + entries + checksum) is written to the `.git/index` file.

**Example Usage:**

```python
from pygit.index import IndexEntry, write_index

# Assume entries is a list of IndexEntry objects, sorted by path
entries = [
    IndexEntry(
        ctime_s=1627890123, ctime_n=0,
        mtime_s=1627890123, mtime_n=0,
        dev=16777220, ino=123456,
        mode=0o100644, uid=1000, gid=1000,
        size=1024,
        sha1=bytes.fromhex('a1b2c3d4e5f67890123456789abcdef012345678'),
        flags=12,
        path='src/main.py'
    ),
    # ... more entries
]

write_index(entries)
print("Index file written with {} entries.".format(len(entries)))
```

---

## ASCII Diagram: Git Index File Structure

```
+--------------------------+
|  Header (12 bytes)       |
|  ----------------------  |
|  Signature: "DIRC" (4B)  |
|  Version: 2 (4B)         |
|  Entry Count (4B)        |
+--------------------------+
|  Entry 1                 |
|  - Fixed 62-byte header  |
|  - Path (variable length)|
|  - Padding (0-7 bytes)   |
+--------------------------+
|  Entry 2                 |
|  - Fixed 62-byte header  |
|  - Path (variable length)|
|  - Padding (0-7 bytes)   |
+--------------------------+
|  ...                    |
+--------------------------+
|  SHA-1 checksum (20 bytes)|
+--------------------------+
```

Each entry looks like this in memory:

```
+-------------------------------------------+
| ctime_s  (4B)                             |
| ctime_n  (4B)                             |
| mtime_s  (4B)                             |
| mtime_n  (4B)                             |
| dev      (4B)                             |
| ino      (4B)                             |
| mode     (4B)                             |
| uid      (4B)                             |
| gid      (4B)                             |
| size     (4B)                             |
| sha1     (20B)                            |
| flags    (2B)                             |
+-------------------------------------------+
| path (variable-length UTF-8 string)       |
| padding (null bytes to multiple of 8B)   |
+-------------------------------------------+
```

---

## Related Functions

- **`read_index()`**  
  Reads and parses the `.git/index` file into `IndexEntry` objects.

- **`add(paths)`**  
  Adds files to the index and internally calls `write_index` after updating entries.

- **`hash_object(data, obj_type, write=True)`**  
  Computes SHA-1 hash and writes object data to the Git object store.

---

This documentation provides a clear understanding of how the Git index file is constructed and persisted by the `write_index` function, enabling users and developers to confidently manipulate the staging area of a Git repository.