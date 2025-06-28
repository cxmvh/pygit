# read_index.md

# Reading Git Index Entries and Interpreting Index File Format

## Overview

The Git index file (commonly `.git/index`) serves as a critical data structure in Git, representing a staging area between the working directory and the repository's object database. This file records metadata and object references for files added to the index, enabling efficient tracking of changes and preparation of commits.

The `read_index.md` document explains how to read and parse Git index entries within the `pygit` project. It details the format of the index file, the interpretation of its contents into `IndexEntry` objects, and how this functionality integrates with other components such as status reporting, adding files, and preparing commits.

Located in the "Index Management" subsection under "Working Copy Status and Diff," this document bridges low-level file parsing with higher-level Git operations like status inspection and committing. Understanding the structure and reading of the index file is essential for correctly implementing features like detecting file changes, preparing commits, and synchronizing with remote repositories.

---

## Important Functions

### 1. `read_index()`

#### Purpose

Reads the Git index file (`.git/index`), parses its binary contents, and returns a list of `IndexEntry` objects representing each tracked file in the index.

This function validates the index file signature and version, verifies the integrity checksum, and iterates over all index entries to extract detailed metadata including timestamps, file modes, object hashes, and paths.

#### Parameters

- None

#### Returns

- `List[IndexEntry]`: A list of `IndexEntry` objects, each encapsulating the data for one indexed file.

#### Preconditions

- The `.git/index` file exists and is accessible.
- The index file format is version 2 (the supported version).
- The SHA-1 checksum at the end of the index file matches the computed digest for integrity.

#### Detailed Explanation of Operation

1. **Read Raw Data**: Attempts to read the entire `.git/index` file as bytes.
2. **Validate Checksum**: Computes the SHA-1 hash over all data except the last 20 bytes, and compares it to the last 20 bytes (stored checksum). Raises assertion error if mismatched.
3. **Parse Header**: Unpacks the first 12 bytes to extract:
   - Signature: should be `b'DIRC'`.
   - Version number: should be `2`.
   - Number of index entries.
4. **Parse Entries**:
   - Iteratively unpacks fixed-length fields (62 bytes) for each entry, including:
     - Creation/modification times (seconds and nanoseconds).
     - Device and inode numbers.
     - File mode.
     - User and group IDs.
     - File size.
     - SHA-1 hash of the blob object.
     - Entry flags.
   - Extracts the variable-length file path string following the fixed fields, terminated by a null byte.
   - Constructs an `IndexEntry` object from all unpacked fields and the path.
   - Advances the cursor by the entry length, rounded up to an 8-byte boundary.
5. **Verify Entry Count**: Confirms that the number of parsed entries matches the header count.
6. **Return Entries**: Returns the list of fully parsed `IndexEntry` objects.

#### Example Usage

```python
from pygit import read_index

# Read and parse the Git index entries
index_entries = read_index()

# Print paths of all files currently tracked in the index
for entry in index_entries:
    print(f"Tracked file: {entry.path} with SHA-1: {entry.sha1.hex()}")
```

---

### 2. `IndexEntry` Data Structure

#### Purpose

Represents a single entry within the Git index, encapsulating all fields parsed from the index file.

#### Fields

- `ctime_s`: Creation time seconds
- `ctime_n`: Creation time nanoseconds
- `mtime_s`: Modification time seconds
- `mtime_n`: Modification time nanoseconds
- `dev`: Device number
- `ino`: Inode number
- `mode`: File mode (permissions and type)
- `uid`: User ID
- `gid`: Group ID
- `size`: File size in bytes
- `sha1`: SHA-1 hash bytes of the file's blob object
- `flags`: Flags for the entry (includes path length and stage bits)
- `path`: File path string

*(Note: The actual data structure definition is elsewhere, but it is returned by `read_index()`.)*

---

### 3. `write_index(entries)`

#### Purpose

Writes a list of `IndexEntry` objects back to the `.git/index` file, serializing the data into the correct binary format for Git.

This complements `read_index()` by allowing updates to the index, such as after adding or removing files.

#### Parameters

- `entries`: List of `IndexEntry` objects to write.

#### Operation Summary

- Serializes each `IndexEntry` into a packed binary form:
  - Packs fixed fields into a 62-byte structure.
  - Encodes the path string, appends null bytes to round the entry size up to an 8-byte boundary.
- Constructs the index header with signature, version, and entry count.
- Concatenates all packed entries.
- Appends a SHA-1 checksum of the entire data.
- Writes the final byte content to `.git/index`.

#### Example Usage

```python
from pygit import read_index, write_index

# Read current index
entries = read_index()

# Modify entries as needed, e.g., remove an entry, add new entries, etc.

# Write updated index back to file
write_index(entries)
```

---

### 4. Integration with Other Functions

- **`get_status()`**: Uses `read_index()` to compare the index contents with the working directory files to detect changed, new, and deleted files.
- **`add(paths)`**: Reads the index, hashes files, creates new `IndexEntry` objects, and writes updated entries using `write_index()`.
- **`commit(message)`**: Relies indirectly on the index being up-to-date, which is managed via `read_index()` and `write_index()`.
- **`diff()`**: Shows differences between index and working copy, relying on reading the index entries.

---

## Git Index File Format — ASCII Diagram

Below is a simplified illustration of the `.git/index` binary format structure parsed by `read_index()`:

```
+------------------------------+
|          Header (12 bytes)    |
|  Signature 'DIRC' (4 bytes)   |
|  Version number (4 bytes)     |
|  Number of entries (4 bytes)  |
+------------------------------+
|        Entry 1 (variable)     |
|  +-------------------------+ |
|  | Fixed fields (62 bytes) | |
|  +-------------------------+ |
|  | Path (variable)         | |
|  +-------------------------+ |
|  | Padding (0-7 bytes)     | |
+------------------------------+
|        Entry 2 (variable)     |
|          ...                  |
+------------------------------+
|        Entry N (variable)     |
+------------------------------+
|    SHA-1 checksum (20 bytes) |
+------------------------------+
```

- Each entry begins with fixed metadata fields (timestamps, mode, size, SHA-1 hash, flags).
- Followed by a null-terminated file path string.
- Each entry is padded with null bytes to align to an 8-byte boundary.
- The file ends with a SHA-1 hash over all preceding bytes to ensure integrity.

---

## Summary

The `read_index.md` document is a fundamental reference for understanding how the `pygit` library reads and interprets the Git index file. The key function `read_index()` parses the binary index file into usable Python objects representing tracked files, enabling other Git operations like status detection, diffing, and committing.

By mastering the index file format and its reading logic, developers can extend or maintain Git-related features in `pygit` with confidence in the correctness of the staging area representation.

---

# Appendix: `read_index()` Function Code Snippet

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

This concludes the technical documentation for `read_index.md`.