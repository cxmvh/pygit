# write_index.md

# Writing Entries Back to the Git Index File

---

## Overview

The `write_index.md` document details the process and functionality for writing entries back to the Git index file (`.git/index`) within the pygit project. The Git index serves as a staging area between the working directory and the repository’s history, holding metadata about files that are about to be committed. This file explains how pygit serializes the in-memory representation of index entries into the correct binary format, ensures data integrity through checksums, and writes this data back to the index file.

This file fits into the broader **Core Commands > Index Management** section of the documentation tree, complementing other index-related files such as `index.md` (which covers reading and manipulating the index) and `read_index.md` (focused on parsing entries). The ability to write to the index is critical for operations like staging files (`add` command) and preparing commit objects, making it a fundamental part of pygit's Git workflow.

---

## Function Documentation

### `write_index(entries)`

#### Purpose

The `write_index` function takes a list of `IndexEntry` objects and writes them to the Git index file in the correct binary format as specified by the Git index file format (version 2). This function ensures that the index file is correctly structured, includes all entry metadata, and ends with a SHA-1 checksum for integrity validation.

#### Parameters

- `entries` (list of `IndexEntry`): A list of index entries representing the current staged files and their metadata. Each `IndexEntry` contains fields such as timestamps, device/inode info, mode, user/group IDs, file size, SHA-1 hash of the blob object, flags, and the file path.

#### Preconditions

- Each `IndexEntry` must be fully populated with valid data.
- Entries should be sorted by the file path to comply with Git index ordering requirements (usually ensured before calling this function).
- The `.git` directory must exist and be writable.

#### Operation Details

1. **Packing Each Entry:**

   For each `IndexEntry` in the input list, the function packs the fixed-length fields into a binary structure using the `struct.pack` format string:

   ```
   '!LLLLLLLLLL20sH'
   ```

   This corresponds to:

   - 4-byte unsigned integers for creation time (seconds and nanoseconds), modification time, device number, inode number, mode, UID, GID, and file size.
   - 20-byte SHA-1 hash of the file blob.
   - 2-byte flags field.
   
2. **Appending the Path:**

   After the fixed-length fields, the file path is appended as a UTF-8 encoded byte string, followed by a null byte (`\x00`).

3. **Padding:**

   To align each entry to an 8-byte boundary, the entry is padded with null bytes. The total length of each entry (including padding) is calculated as:

   ```
   length = ((62 + len(path) + 8) // 8) * 8
   ```
   
   Here, `62` is the size of the fixed-length packed fields.

4. **Constructing the Index File Header:**

   The index file starts with a 12-byte header consisting of:

   - Signature: `b'DIRC'` (4 bytes)
   - Version number: 2 (4 bytes)
   - Number of entries (4 bytes)

5. **Writing the File:**

   The header and all packed entries are concatenated. A SHA-1 checksum of this entire content is computed and appended to the end of the file to ensure integrity.

6. **Saving the File:**

   The final byte sequence is written to the `.git/index` file using the utility function `write_file`.

#### Example Usage

```python
from pygit import read_index, write_index

# Read current index entries
entries = read_index()

# Modify entries in some way, for example, removing a file or adding a new one
# (This example assumes entries are already updated)

# Write back the updated index entries
write_index(entries)
```

---

## ASCII Diagram: Index Entry Structure

```
+-----------------+------------------+------------------+
| Fixed-Length    | File Path        | Padding          |
| Fields (62 B)   | (variable length)| (0 to 7 nulls)   |
+-----------------+------------------+------------------+
| ctime_s (4 B)   |                  |                  |
| ctime_n (4 B)   |                  |                  |
| mtime_s (4 B)   |                  |                  |
| mtime_n (4 B)   |                  |                  |
| dev (4 B)       |                  |                  |
| ino (4 B)       |                  |                  |
| mode (4 B)      |                  |                  |
| uid (4 B)       |                  |                  |
| gid (4 B)       |                  |                  |
| size (4 B)      |                  |                  |
| sha1 (20 B)     |                  |                  |
| flags (2 B)     |                  |                  |
+-----------------+------------------+------------------+
```

Each entry is aligned to 8 bytes by adding null bytes after the path.

---

## Summary

The `write_index` function is a core utility for pygit's index management, enabling the serialization of staged file metadata into the Git index format. This operation is essential for staging changes and preparing commits. Proper indexing ensures that subsequent commands such as `commit` and `ls-files` operate on accurate and consistent data.

By understanding the structure and functionality provided here, developers and users of pygit can modify, extend, or debug index writing operations confidently.

---

# End of `write_index.md` documentation file content.