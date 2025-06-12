# read_index.md

# Reading the Git Index File (`read_index`)

## Overview

The `read_index.md` document details the implementation and usage of the `read_index` function, which is responsible for reading Git's index file (`.git/index`). The Git index is a critical data structure that acts as a staging area between the working directory and the repository's commit history. This file plays an essential role in managing the state of the working copy, tracking changes, and preparing files for commit.

Within the broader project documentation, this file is situated under the **Index and Working Copy Management** section, which encompasses operations like reading, writing, and manipulating the Git index and working copy files. The `read_index` function is foundational for other operations such as `diff`, `status`, `add`, and `commit`, all of which rely on accurate interpretation of the index file to function correctly.

---

## Function: `read_index()`

### Purpose

The `read_index` function reads the Git index file (`.git/index`) and parses its contents into a list of structured `IndexEntry` objects. Each `IndexEntry` represents a file tracked in the index, including metadata such as timestamps, file mode, SHA-1 hash of the contents, and the path.

This function is crucial for any operation that requires knowledge of the current index state, such as showing diffs, checking status, or preparing commits.

### Parameters

- None

### Returns

- A list of `IndexEntry` objects representing the entries in the index file.
- Returns an empty list if the index file does not exist.

### Preconditions

- The `.git` directory must exist in the current working directory.
- The index file at `.git/index` should conform to the Git index file format version 2.

### How It Works: Step-by-Step

1. **Open and Read Index File**  
   Attempts to read the contents of `.git/index` as a byte stream. If the file is missing, the function returns an empty list, indicating no index entries.

2. **Validate Checksum**  
   The last 20 bytes of the index file represent a SHA-1 checksum. The function computes the SHA-1 hash of the data excluding these last 20 bytes and asserts that it matches the stored checksum. This ensures data integrity.

3. **Parse Header**  
   The first 12 bytes of the file include:
   - Signature: 4 bytes, expected to be `b'DIRC'`.
   - Version: 4-byte unsigned integer, expected to be `2`.
   - Number of entries: 4-byte unsigned integer indicating how many index entries follow.

4. **Parse Index Entries**  
   The function proceeds to parse each index entry sequentially:
   - Each entry starts with a fixed 62-byte header containing metadata fields such as:
     - Creation and modification times
     - Device and inode numbers
     - File mode
     - User and group IDs
     - File size
     - SHA-1 hash of the file contents
     - Flags
   - Following the header is a null-terminated path string.
   - The total entry length is padded to a multiple of 8 bytes to maintain alignment.
   - This process repeats until all entries are parsed.

5. **Return Entries**  
   After parsing all entries, the function returns a list of `IndexEntry` objects.

---

### Example Usage

```python
from pygit import read_index

# Read the current index entries
index_entries = read_index()

# Print all tracked file paths
for entry in index_entries:
    print(f'Path: {entry.path}, SHA-1: {entry.sha1.hex()}')
```

---

### Code Snippet of `read_index`

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

### `IndexEntry` Structure Overview

Each `IndexEntry` typically contains the following fields:

| Field       | Description                          |
|-------------|------------------------------------|
| ctime_s     | Creation time seconds               |
| ctime_n     | Creation time nanoseconds           |
| mtime_s     | Modification time seconds           |
| mtime_n     | Modification time nanoseconds       |
| dev         | Device number                      |
| ino         | Inode number                      |
| mode        | File mode (permissions and type)   |
| uid         | User ID                           |
| gid         | Group ID                          |
| size        | File size in bytes                 |
| sha1        | SHA-1 hash of the file contents    |
| flags       | Flags (includes stage number etc.) |
| path        | File path relative to repository root |

---

### ASCII Diagram: Git Index File Structure

```
+----------------+----------------+----------------+----------------+----------------+
| Header (12 b)  | Entry 1        | Entry 2        | ...            | Checksum (20 b)|
|----------------|----------------|----------------|----------------|----------------|
| Signature:4 b  | Entry Header   | Entry Header   |                | SHA-1 Digest   |
| Version:4 b    | Path (null-    | Path (null-    |                | of preceding   |
| Num Entries:4 b| terminated)    | terminated)    |                | data           |
+----------------+----------------+----------------+----------------+----------------+

Each Entry:
+-----------------------------------------------+
| Fixed 62-byte header fields                     |
+-----------------------------------------------+
| Null-terminated path string                     |
+-----------------------------------------------+
| Padding to next 8-byte boundary                  |
+-----------------------------------------------+
```

---

## Integration with Other Components

- **Used by `diff`**: The `diff()` function reads the index entries via `read_index()` to compare the current index state with the working copy.

- **Used by `status`**: Retrieves index entries to determine changed, new, or deleted files.

- **Used by `add`**: Reads existing index entries to update or add new files.

- **Used by `write_index`**: After modifications, the updated list of `IndexEntry` objects can be written back to `.git/index`.

---

## Summary

The `read_index` function is a foundational utility for interacting with Git's staging area. By parsing the binary index file into manageable `IndexEntry` objects, it enables higher-level Git operations to inspect and manipulate the repository's state effectively.

Understanding the structure and parsing logic of the index file is critical for anyone looking to work with Git internals, build Git-related tools, or contribute to the `pygit` project.