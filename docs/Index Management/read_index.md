# read_index.md

# Documentation for Reading the Git Index File

---

## Overview

The `read_index.md` documentation details the process and functionality involved in reading the Git index file, a core component of Git's internal data management system. The Git index (also known as the staging area) acts as a bridge between the working directory and the repository, tracking files slated for the next commit, their metadata, and content hashes. This document is part of the **Index Management** section, which covers reading, writing, and managing the Git index. Understanding how to read the index file is crucial for implementing features that inspect the staged files, determine changes, or prepare commits.

The `read_index` function presented here reads the `.git/index` file, parses its binary format including file metadata and paths, validates the file integrity via checksums, and returns a structured list of index entries. This foundational operation supports higher-level workflows like `git diff`, `git status`, and committing changes.

---

## Function Documentation

### `read_index()`

#### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects, each representing a tracked file along with its metadata (timestamps, device/inode numbers, file mode, ownership, size, SHA-1 blob hash, and flags). It validates the index file's signature, version, and checksum to ensure data integrity.

This function is essential for any operation that needs to inspect the current state of the index, such as detecting changes compared to the working directory or preparing data for commit objects.

#### Parameters

- None

#### Returns

- `List[IndexEntry]`: A list of `IndexEntry` instances representing each entry in the Git index.

#### Preconditions

- The `.git/index` file must exist and be readable.
- The index file must conform to version 2 format.
- The checksum (SHA-1) at the end of the index file must be valid.

If the index file is missing, an empty list is returned.

#### Operation Details

1. **Read Index File Data**  
   The function attempts to read the raw bytes of `.git/index`. If the file is not found, it returns an empty list, indicating no staged files.

2. **Checksum Verification**  
   The last 20 bytes of the file represent a SHA-1 checksum of the preceding bytes. The function computes the SHA-1 digest of the file content excluding the checksum and compares it to the stored checksum to verify integrity.

3. **Header Parsing**  
   The index file header consists of:
   - Signature: 4 bytes, expected to be `b'DIRC'`
   - Version: 4-byte unsigned integer, expected to be `2`
   - Number of entries: 4-byte unsigned integer

4. **Entry Extraction**  
   Following the header, the file contains multiple index entries. Each entry includes fixed-length metadata fields (62 bytes), followed by a variable-length path string terminated by a null byte (`\x00`), and padded to a multiple of 8 bytes for alignment.

   For each entry:
   - Unpack the fixed metadata fields using `struct.unpack`.
   - Extract the null-terminated path string.
   - Create an `IndexEntry` object with all fields.
   - Move to the next entry by advancing the index pointer by the entry's padded length.

5. **Return Entries**  
   After processing all entries, the function asserts that the number of parsed entries matches the header count and returns the list.

#### Example Usage

```python
from pygit import read_index

index_entries = read_index()

for entry in index_entries:
    print(f"Path: {entry.path}")
    print(f"  SHA-1: {entry.sha1.hex()}")
    print(f"  Mode: {oct(entry.mode)}")
    print(f"  Size: {entry.size} bytes")
    print()
```

This example reads the Git index and prints details about each staged file, including its path, SHA-1 hash, file mode, and size.

---

## Supporting Concepts Illustrated

### Git Index File Structure (Simplified)

```
+----------------------+----------------+-------------------+----------------------+
| Header (12 bytes)     | Index Entries  | SHA-1 Checksum (20 bytes)             |
|----------------------|----------------|--------------------------------------|
| Signature: "DIRC"     | Entry 1        |                                      |
| Version: 2            | Entry 2        |                                      |
| Entry count (N)       | ...            |                                      |
+----------------------+----------------+-------------------+----------------------+

Each Index Entry:
+-------------------------------+-----------------------------+
| Fixed metadata (62 bytes)      | Path (variable length + \0) |
| (ctime, mtime, dev, ino, mode, | Padded with null bytes to    |
| uid, gid, size, SHA-1, flags)  | 8-byte alignment            |
+-------------------------------+-----------------------------+
```

### Index Entry Parsing Loop (ASCII Diagram)

```
+------------------+-------------------+------------------+------------------+
| Bytes 0..61      | Bytes 62..(N)     | Next Entry Start  | ...              |
| Fixed fields     | Path (null-term.) | (aligned to 8)   |                  |
+------------------+-------------------+------------------+------------------+
      |                     |                     |
      |-- unpack fixed ----->|-- read path -------->|
                                (ends at \0)
```

---

## Summary

The `read_index()` function is a vital utility in the Git implementation stack, enabling reading and interpreting the Git index file. It ensures the file is well-formed, parses entries reliably, and provides structured data about the staged files. This facilitates diffing, status checking, and committing workflows, making it a foundational component of Git's internal operations.

For further details on index management, see related documentation files:
- [index.md](index.md): Comprehensive handling of index read/write.
- [write_index.md](write_index.md): Writing entries back to the index.
- [add.md](add.md): Adding files to the index.

---

# Appendix: Code Snippet of `read_index()`

```python
def read_index():
    """Read git index file and return list of IndexEntry objects."""
    try:
        data = read_file(os.path.join('.git', 'index'))
    except FileNotFoundError:
        return []
    digest = hashlib.sha1(data[:-20]).digest()
    assert digest == data[-20:], 'invalid index checksum'
    signature, version, num_entries = struct.unpack('!4sLL', data[:12])
    assert signature == b'DIRC', 'invalid index signature {}'.format(signature)
    assert version == 2, 'unknown index version {}'.format(version)
    entry_data = data[12:-20]
    entries = []
    i = 0
    while i + 62 < len(entry_data):
        fields_end = i + 62
        fields = struct.unpack('!LLLLLLLLLL20sH', entry_data[i:fields_end])
        path_end = entry_data.index(b'\x00', fields_end)
        path = entry_data[fields_end:path_end]
        entry = IndexEntry(*(fields + (path.decode(),)))
        entries.append(entry)
        entry_len = ((62 + len(path) + 8) // 8) * 8
        i += entry_len
    assert len(entries) == num_entries
    return entries
```

---

# IndexEntry Data Structure (for reference)

```python
from collections import namedtuple

IndexEntry = namedtuple('IndexEntry', [
    'ctime_s', 'ctime_n', 'mtime_s', 'mtime_n', 'dev', 'ino',
    'mode', 'uid', 'gid', 'size', 'sha1', 'flags', 'path'
])
```

This structure holds all metadata parsed from each index entry.

---

# End of `read_index.md` Documentation