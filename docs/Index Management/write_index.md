# write_index.md

# Documentation for Writing the Git Index File

---

## Overview

The `write_index.md` document provides detailed technical documentation on how the Git index file is written and managed within the repository. The Git index serves as a staging area that tracks changes to files before committing them to the repository. This file is critical in coordinating the state between the working directory and the Git repository’s history.

Situated within the **Index Management** section of the broader Git documentation tree, this file complements related documentation such as reading the index (`read_index.md`) and listing indexed files (`ls_files.md`). It focuses specifically on the process of serializing and writing a collection of index entries to the `.git/index` file, ensuring data integrity and compatibility with Git's expected format.

This document is essential for developers and maintainers working on Git internals, especially those implementing or modifying how file metadata and states are recorded prior to commit operations.

---

## Function Documentation

### `write_index(entries)`

#### Purpose

The `write_index` function serializes a list of `IndexEntry` objects and writes them into the Git index file (`.git/index`). This function ensures that the index file conforms to Git’s binary format specification, including a header, multiple index entries, and a SHA-1 checksum for data integrity.

#### Parameters

- `entries` (`List[IndexEntry]`): A list of `IndexEntry` objects to be written to the index file. Each entry represents a file tracked in the index, including metadata such as timestamps, device and inode numbers, file mode, user and group IDs, file size, SHA-1 hash of file content, and flags (including path length).

#### Preconditions

- Each `IndexEntry` must be correctly populated with valid data.
- The list `entries` should be sorted by file path for proper indexing.
- The `.git` directory must exist and be writable.
- The index file will be overwritten with the new data.

#### Operation Steps

1. **Pack Each Entry:**

   Each `IndexEntry` is packed into a binary format using `struct.pack` with the following fields:

   - `ctime_s` (creation time seconds)
   - `ctime_n` (creation time nanoseconds)
   - `mtime_s` (modification time seconds)
   - `mtime_n` (modification time nanoseconds)
   - `dev` (device number)
   - `ino` (inode number)
   - `mode` (file mode)
   - `uid` (user ID)
   - `gid` (group ID)
   - `size` (file size)
   - `sha1` (20-byte SHA-1 hash of the file content)
   - `flags` (various flags, including path length)

   This header is 62 bytes in length.

2. **Add Path and Pad:**

   - The file path is encoded as bytes and appended after the 62-byte header.
   - A null byte (`b'\x00'`) terminates the path.
   - The entire entry is padded with null bytes to ensure the entry is aligned to an 8-byte boundary. The length is calculated as:

     ```
     length = ((62 + len(path) + 8) // 8) * 8
     ```

3. **Build Index Header:**

   - The index header consists of:
     - Signature: 4 bytes, ASCII string `'DIRC'`
     - Version number: 4 bytes, unsigned integer (currently `2`)
     - Number of entries: 4 bytes, unsigned integer
   - The header is packed using `struct.pack('!4sLL', b'DIRC', 2, len(entries))`.

4. **Concatenate All Entries:**

   - All packed entries are concatenated after the header.

5. **Calculate SHA-1 Checksum:**

   - Compute the SHA-1 hash of all data except the checksum itself.
   - Append the 20-byte SHA-1 checksum at the end of the file.

6. **Write to Index File:**

   - The final concatenated data (header + entries + checksum) is written to `.git/index` using a helper function.

#### Example Usage

```python
from collections import namedtuple
import os
import hashlib

# Assume IndexEntry is a namedtuple matching the fields:
IndexEntry = namedtuple('IndexEntry', [
    'ctime_s', 'ctime_n', 'mtime_s', 'mtime_n', 'dev', 'ino',
    'mode', 'uid', 'gid', 'size', 'sha1', 'flags', 'path'
])

# Example: create a single index entry for a file "example.txt"
st = os.stat('example.txt')
with open('example.txt', 'rb') as f:
    data = f.read()
sha1 = hashlib.sha1(data).digest()

entry = IndexEntry(
    ctime_s=int(st.st_ctime),
    ctime_n=0,
    mtime_s=int(st.st_mtime),
    mtime_n=0,
    dev=st.st_dev,
    ino=st.st_ino,
    mode=st.st_mode,
    uid=st.st_uid,
    gid=st.st_gid,
    size=st.st_size,
    sha1=sha1,
    flags=len('example.txt'.encode('utf-8')),
    path='example.txt'
)

# Write index with just this entry
write_index([entry])
```

---

## Detailed Code Explanation

```python
def write_index(entries):
    """Write list of IndexEntry objects to git index file."""
    packed_entries = []
    for entry in entries:
        # Pack the fixed-size fields of the entry into binary format
        entry_head = struct.pack('!LLLLLLLLLL20sH',
                entry.ctime_s, entry.ctime_n, entry.mtime_s, entry.mtime_n,
                entry.dev, entry.ino, entry.mode, entry.uid, entry.gid,
                entry.size, entry.sha1, entry.flags)
        
        # Encode path and calculate total length with padding to 8-byte boundary
        path = entry.path.encode()
        length = ((62 + len(path) + 8) // 8) * 8
        
        # Create the full packed entry with null padding
        packed_entry = entry_head + path + b'\x00' * (length - 62 - len(path))
        packed_entries.append(packed_entry)
    
    # Pack the index file header: signature, version, number of entries
    header = struct.pack('!4sLL', b'DIRC', 2, len(entries))
    
    # Concatenate header and all entries
    all_data = header + b''.join(packed_entries)
    
    # Compute SHA-1 checksum of all data except checksum itself
    digest = hashlib.sha1(all_data).digest()
    
    # Write the full index file including checksum
    write_file(os.path.join('.git', 'index'), all_data + digest)
```

---

## ASCII Diagram: Git Index File Layout

```
+---------------------+
|  Index Header       |  <- Signature 'DIRC' (4 bytes)
|                     |  <- Version (4 bytes)
|                     |  <- Number of entries (4 bytes)
+---------------------+
|  Index Entry #1     |  <- Fixed 62 bytes + path + padding
+---------------------+
|  Index Entry #2     |
+---------------------+
|        ...          |
+---------------------+
|  Index Entry #N     |
+---------------------+
|  SHA-1 Checksum     |  <- 20 bytes SHA-1 of all previous bytes
+---------------------+
```

Each **Index Entry** contains detailed metadata about a single file tracked by Git.

---

## Related Functions

For further understanding and usage context, see related functions:

- **`read_index()`**: Reads and parses the `.git/index` file into `IndexEntry` objects.
- **`add(paths)`**: Adds files to the index by creating and updating `IndexEntry` objects, then calls `write_index`.
- **`hash_object(data, obj_type, write=True)`**: Hashes file content to generate SHA-1 used in index entries.
- **`write_file(path, data)`**: Helper to write bytes data to disk.
- **`ls_files(details=False)`**: Lists files currently in the index, useful for verifying index content.

---

## Summary

The `write_index` function is a critical component that serializes the staged file metadata and writes it to the `.git/index` file in a format compatible with Git internals. Understanding this process is fundamental to grasping how Git maintains the staging area and prepares for commits. This document serves as a reference for implementing or debugging Git index writing functionality.