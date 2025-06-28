# write_index.md

# Writing entries to the Git index file

---

## Overview

This document details the functionality for writing entries to the Git index file within the `pygit` project. The Git index (also called the staging area) is a critical component that tracks changes between the working directory and the repository. This file focuses on the `write_index` function, which serializes a list of `IndexEntry` objects back into the `.git/index` file in the correct Git index format.

The `write_index` function is part of the broader **Index Management** subsystem documented under the **Working Copy Status and Diff** section of the overall project. This subsystem includes reading (`read_index`), writing (`write_index`), adding files (`add`), and listing indexed files (`ls_files`). Proper management of the Git index ensures accurate staging and committing of file changes.

Understanding `write_index` is essential for developers working on staging functionality, index manipulation, and any features that require modifying the index contents programmatically.

---

## Function: `write_index(entries)`

### Purpose

The `write_index` function takes a list of `IndexEntry` objects and writes them to the Git index file (`.git/index`) according to the Git index file format specification. This involves serializing each entry's metadata and path, computing the overall checksum, and writing the binary data atomically.

### Parameters

- `entries`: A list of `IndexEntry` objects. Each `IndexEntry` represents a file or directory entry with metadata such as creation and modification times, device and inode numbers, file mode, user/group IDs, file size, SHA-1 hash of the content, flags, and the file path.

### Preconditions

- Each element in `entries` must be an instance of `IndexEntry` with all fields properly populated.
- The entries list should be sorted by path (commonly done before calling this function to maintain Git's index order).
- The `.git/` directory should exist and be accessible for writing.

### Operation Details

1. **Entry Serialization:**

   - Each `IndexEntry` is serialized to a binary format using a fixed header layout of 62 bytes followed by the variable-length file path and null padding.
   - The fixed part includes fields such as:
     - Creation time (seconds and nanoseconds)
     - Modification time (seconds and nanoseconds)
     - Device number, inode number
     - File mode (permissions and type)
     - User ID, group ID
     - File size
     - SHA-1 hash (20 bytes)
     - Flags (2 bytes)
   - The file path is encoded as UTF-8 bytes and appended after the header, followed by null (`\x00`) bytes to pad the entry to an 8-byte boundary (as required by Git index format).

2. **Index File Header:**

   - The header starts with a 4-byte signature `b'DIRC'`.
   - Followed by a 4-byte version number (value `2`).
   - Followed by a 4-byte count of entries.

3. **Checksum Calculation:**

   - The entire data (header + all serialized entries) is SHA-1 hashed.
   - The SHA-1 digest (20 bytes) is appended at the end of the index file, serving as a checksum to verify data integrity when reading back.

4. **Writing to File:**

   - The complete binary blob (header + entries + checksum) is written to `.git/index` atomically.
   - Uses the utility function `write_file` to write the bytes.

---

### Example Usage

```python
from collections import namedtuple
import os

# Assuming IndexEntry is a namedtuple or class with the following fields:
IndexEntry = namedtuple('IndexEntry', [
    'ctime_s', 'ctime_n', 'mtime_s', 'mtime_n',
    'dev', 'ino', 'mode', 'uid', 'gid',
    'size', 'sha1', 'flags', 'path'
])

# Example: Create a sample IndexEntry for a file 'example.txt'
entry = IndexEntry(
    ctime_s=1625097600,       # creation time seconds
    ctime_n=0,               # creation time nanoseconds
    mtime_s=1625097600,       # modification time seconds
    mtime_n=0,               # modification time nanoseconds
    dev=2049,                # device number
    ino=305419896,           # inode number
    mode=0o100644,           # regular file with permissions
    uid=1000,                # user id
    gid=1000,                # group id
    size=123,                # file size in bytes
    sha1=bytes.fromhex('e69de29bb2d1d6434b8b29ae775ad8c2e48c5391'),  # SHA-1 hash of file contents
    flags=len('example.txt'),  # length of path (for flags)
    path='example.txt'       # file path
)

# Write this single entry to the index
write_index([entry])
```

---

### Detailed Code Explanation

```python
def write_index(entries):
    """Write list of IndexEntry objects to git index file."""
    packed_entries = []
    for entry in entries:
        # Pack fixed-size fields (62 bytes total) according to Git index format:
        # ! - network byte order (big-endian)
        # L - unsigned long (4 bytes)
        # 10 fields for times, device, inode, mode, uid, gid, size
        # 20s - 20 bytes for SHA-1 hash
        # H - unsigned short (2 bytes) for flags
        entry_head = struct.pack(
            '!LLLLLLLLLL20sH',
            entry.ctime_s, entry.ctime_n,
            entry.mtime_s, entry.mtime_n,
            entry.dev, entry.ino,
            entry.mode, entry.uid, entry.gid,
            entry.size,
            entry.sha1,
            entry.flags
        )
        path = entry.path.encode()  # Encode path to bytes
        # Calculate padding so total entry size is multiple of 8 bytes
        length = ((62 + len(path) + 8) // 8) * 8
        padding = b'\x00' * (length - 62 - len(path))

        packed_entry = entry_head + path + padding
        packed_entries.append(packed_entry)

    # Build index header: signature 'DIRC', version 2, number of entries
    header = struct.pack('!4sLL', b'DIRC', 2, len(entries))

    all_data = header + b''.join(packed_entries)

    # Compute SHA-1 checksum of all data (excluding the checksum itself)
    digest = hashlib.sha1(all_data).digest()

    # Write the index file with checksum appended
    write_file(os.path.join('.git', 'index'), all_data + digest)
```

---

## Additional Notes

- **Index Format Alignment:** The padding ensures that each entry's size aligns to an 8-byte boundary. This aligns with the official Git index file format specification and is critical for compatibility.

- **Checksum Verification:** When reading the index, the checksum is verified to ensure data integrity.

- **Integration:** `write_index` is typically called after new files are added or existing files are modified in the index, e.g., in the `add()` function which hashes file contents and creates new `IndexEntry` objects.

---

## ASCII Diagram: Git Index File Structure

```
+--------------------+
| Header             |
| - Signature 'DIRC'  | 4 bytes
| - Version (2)       | 4 bytes
| - Number of entries | 4 bytes
+--------------------+
| Entry 1            | variable length (multiple of 8 bytes)
| - Fixed 62 bytes    |
| - Path (variable)  |
| - Null padding     |
+--------------------+
| Entry 2            | ...
+--------------------+
| ...                |
+--------------------+
| SHA-1 checksum     | 20 bytes (checksum of all previous bytes)
+--------------------+
```

---

# Summary

The `write_index` function is a fundamental component for updating the Git index file from `IndexEntry` objects. It handles binary serialization, header construction, padding for alignment, checksum calculation, and writing to disk. Proper use of this function ensures the Git index remains a consistent and reliable staging area for commits within the `pygit` system.