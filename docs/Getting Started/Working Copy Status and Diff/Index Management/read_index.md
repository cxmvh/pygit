# read_index.md

# Reading the Git Index File and Interpreting IndexEntry Objects

---

## Overview

The `read_index.md` document covers the critical functionality of reading the Git index file (`.git/index`) and interpreting its contents as `IndexEntry` objects. The Git index is a binary file that acts as a staging area, tracking metadata and blob references for files intended for the next commit.

Within the broader **Git Index and Working Directory** section of the pygit documentation, this file focuses on the low-level parsing of the index file format, which is essential for commands like `status`, `diff`, and `add`. Understanding how to read the index enables pygit to monitor changes between the working copy and the repository, and to prepare data for commits.

---

## Key Function: `read_index()`

### Purpose

`read_index()` reads the `.git/index` file and returns a list of `IndexEntry` objects representing the files staged in the index. Each `IndexEntry` contains metadata such as timestamps, device/inode numbers, file mode, user/group IDs, file size, SHA-1 hash, and path.

This function is foundational for higher-level operations that manipulate or query the index.

### Parameters

- None

### Returns

- `List[IndexEntry]`: A list of `IndexEntry` instances parsed from the index file.

### Preconditions

- The `.git/index` file must exist and be a valid Git index version 2 file.
- The file must have a correct SHA-1 checksum at the end.

### How It Works

1. **Read the Index File Bytes**
   - Attempts to read `.git/index` file contents as raw bytes.
   - If the file does not exist, returns an empty list.

2. **Verify Checksum**
   - Computes the SHA-1 hash of all bytes except the last 20 bytes.
   - Compares this digest to the last 20 bytes of the file to ensure data integrity.

3. **Parse Header**
   - Unpacks the first 12 bytes to extract:
     - Signature: Should be `b'DIRC'`.
     - Version number: Must be `2`.
     - Number of entries.

4. **Parse Each Entry**
   - The remainder of the file (excluding the trailing checksum) contains multiple entries.
   - Each entry begins with fixed-length fields (62 bytes), unpacked into various metadata.
   - The file path string follows, terminated by a null byte.
   - The entry is padded to an 8-byte boundary.
   - Constructs an `IndexEntry` object for each entry and appends it to the list.

5. **Return the List**
   - Verifies the number of entries matches the header count.
   - Returns the list of parsed entries.

### Example Usage

```python
from pygit import read_index

def print_index_paths():
    entries = read_index()
    print("Files staged in index:")
    for entry in entries:
        print(f" - {entry.path} (SHA-1: {entry.sha1.hex()})")

print_index_paths()
```

---

## Supporting Concepts and Data Structures

### IndexEntry Object Structure

Each `IndexEntry` represents one file in the index and contains the following fields:

```
+----------------+------------------------------+
| Field          | Description                  |
+----------------+------------------------------+
| ctime_s        | Creation time (seconds)       |
| ctime_n        | Creation time (nanoseconds)   |
| mtime_s        | Modification time (seconds)   |
| mtime_n        | Modification time (nanoseconds)|
| dev            | Device number                 |
| ino            | Inode number                 |
| mode           | File mode (permissions/type) |
| uid            | User ID                      |
| gid            | Group ID                     |
| size           | File size                    |
| sha1           | SHA-1 hash of file content   |
| flags          | Flags (including path length)|
| path           | Path string (file path)      |
+----------------+------------------------------+
```

### Index File Binary Format Overview

The `.git/index` file structure:

```
+------------+------------+---------------------+
| Offset     | Size       | Description         |
+------------+------------+---------------------+
| 0          | 4 bytes    | Signature: "DIRC"   |
| 4          | 4 bytes    | Version number (2)  |
| 8          | 4 bytes    | Number of entries   |
| 12         | variable   | Entries             |
| ...        |            |                     |
| EOF-20     | 20 bytes   | SHA-1 checksum      |
+------------+------------+---------------------+
```

Each entry:

```
+-----------------+----------------------+
| Field           | Size (bytes)         |
+-----------------+----------------------+
| ctime_s         | 4                    |
| ctime_n         | 4                    |
| mtime_s         | 4                    |
| mtime_n         | 4                    |
| dev             | 4                    |
| ino             | 4                    |
| mode            | 4                    |
| uid             | 4                    |
| gid             | 4                    |
| size            | 4                    |
| sha1            | 20                   |
| flags           | 2                    |
| path            | variable (null-terminated) |
| padding         | 0-7 bytes to 8-byte alignment |
+-----------------+----------------------+
```

---

## ASCII Diagram: Index File Layout

```
+------------------------------------------------+
| Signature "DIRC" (4 bytes)                      |
+------------------------------------------------+
| Version number (4 bytes)                        |
+------------------------------------------------+
| Number of entries (4 bytes)                     |
+------------------------------------------------+
| Entry 1                                         |
|   - Fixed fields (62 bytes)                     |
|   - Path (null-terminated)                      |
|   - Padding to align next entry to 8-byte boundary |
+------------------------------------------------+
| Entry 2                                         |
|   ...                                           |
+------------------------------------------------+
| ...                                             |
+------------------------------------------------+
| SHA-1 checksum of all previous bytes (20 bytes)|
+------------------------------------------------+
```

---

## Related Functions and Usage Context

- **`read_index()`** is used by `get_status()`, `status()`, `diff()`, and `add()` functions to understand the current staged files and their metadata.
- The entries returned by `read_index()` are manipulated or compared to working directory files to detect changes or prepare commits.
- The companion function `write_index()` enables writing updated `IndexEntry` lists back to the index file after modifications.

---

## Summary

`read_index()` provides a direct interface to parse the Git index file into structured `IndexEntry` objects. Its correctness and efficiency are crucial for providing accurate status reporting, adding files to staging, and preparing commits in pygit. Understanding this function is foundational for any further index manipulation and Git operations within the pygit framework.