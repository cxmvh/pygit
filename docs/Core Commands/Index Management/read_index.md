# read_index.md

# Reading the Git Index File and Parsing Entries

---

## Overview

The `read_index.md` document explains the process of reading and parsing the Git index file (`.git/index`). The Git index acts as a staging area, keeping track of files and their metadata prior to committing. This file plays a critical role in Git’s internal mechanics by maintaining a snapshot of the working directory state.

Within the broader documentation tree, this file is part of the **Core Commands > Index Management** section, which covers operations related to the Git index, including reading, writing, and manipulating index entries. The functionality described here underpins commands such as `pygit.ls_files` and is foundational for other operations like status reporting, committing, and diffing.

Understanding how to read and parse the index file is essential for implementing Git commands that rely on the index state, making this document a key reference for developers working with Git internals or building Git-like tools.

---

## Function Documentation

### Function: `read_index()`

#### Purpose

Reads the Git index file located at `.git/index` and parses it into a list of `IndexEntry` objects representing each file entry staged in the index.

The function verifies the integrity and format of the index file, including validating the checksum and ensuring the file signature and version are as expected.

#### Parameters

- None.

#### Returns

- A list of `IndexEntry` objects, each containing metadata such as file timestamps, device and inode info, file mode, ownership, file size, SHA-1 hash of the blob, flags, and the file path.

#### Preconditions

- The `.git/index` file must exist and be accessible.
- The index file must conform to Git index version 2 format.
- The SHA-1 checksum at the end of the index file must be valid, otherwise an assertion error is raised.

#### Operation Details

1. Attempts to read the raw bytes of `.git/index`.
2. If the file does not exist, returns an empty list (indicating no index entries).
3. Validates the SHA-1 checksum of the index data (all but the last 20 bytes).
4. Unpacks the header:
   - Verifies the signature is `DIRC`.
   - Confirms the index version is 2.
   - Reads the number of entries.
5. Parses each index entry sequentially:
   - Each entry consists of fixed-size fields (62 bytes) followed by a null-terminated path string.
   - Extracts the fields using `struct.unpack` with format `!LLLLLLLLLL20sH`.
   - Reads the path string terminated by a null byte.
   - Computes the padded length of each entry to align to an 8-byte boundary.
   - Advances the parsing index accordingly.
6. Asserts that the number of parsed entries matches the expected count.
7. Returns the fully parsed list of `IndexEntry` objects.

#### Example Usage

```python
from pygit import read_index

# Read the current Git index entries
entries = read_index()

# Print the paths of all files in the index
for entry in entries:
    print(f"Staged file: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

#### ASCII Diagram: Structure of `.git/index` File

```
+-------------------+----------------+-------------------------+------------------+
| Header (12 bytes)  | Entries        | SHA-1 Checksum (20 bytes)|
+-------------------+----------------+-------------------------+------------------+
| Signature (4 bytes)| Entry 1        | Entry 2                 | ...              |
| Version (4 bytes)  | ...            |                         |                  |
| Num Entries (4 b)  | (variable size)|                         |                  |
+-------------------+----------------+-------------------------+------------------+

Each Entry:
+--------------------------------------------------------------+
| 62 bytes fixed fields | Path (null-terminated) | Padding    |
+--------------------------------------------------------------+
| ctime, mtime, dev...  | file path string        | 0 to 7 0s  |
+--------------------------------------------------------------+
(Entry length aligned to 8 bytes)
```

---

## Related Functions

While `read_index()` is the core function for reading the index, it is often used in conjunction with:

- **`ls_files(details=False)`**: Lists files in the index, optionally showing details like mode, SHA-1, and stage.
- **`write_index(entries)`**: Writes a list of `IndexEntry` objects back to the `.git/index` file.
- **`add(paths)`**: Adds files to the index by updating entries.
- **`get_status()`**: Compares the index entries to the working directory to detect changes.

---

## Summary

The `read_index()` function is vital for accessing the Git index state. It carefully parses the `.git/index` file, validates its structure and checksum, and returns a structured list of index entries. This enables higher-level Git commands to interact with the staged snapshot of the repository efficiently.

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
    assert signature == b'DIRC', \
            'invalid index signature {}'.format(signature)
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

# End of `read_index.md` Documentation