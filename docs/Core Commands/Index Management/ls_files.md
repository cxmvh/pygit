# ls_files.md

## Overview

The `ls_files.md` file documents the functionality for listing files tracked in the Git index within the `pygit` project. It focuses on the `ls_files` command, which outputs the list of files currently staged in the Git index, optionally including detailed information such as file mode, SHA-1 hash, and stage number. This file is part of the **Core Commands > Index Management** section, which handles operations related to the Git index, including reading, writing, and managing file entries. Understanding this command is essential for users and developers who want to inspect the state of the index and verify which files are staged for commit.

---

## Function Documentation

### `ls_files(details=False)`

Prints the list of files recorded in the Git index. When called without arguments, it prints only the file paths. If `details=True`, it prints additional metadata for each file entry: the file mode (permissions), the SHA-1 hash of the file content, and the stage number (used for conflict resolution).

#### Parameters

- `details` (bool, optional): When `True`, prints detailed information about each indexed file entry. Defaults to `False`.

#### Description

This function reads the Git index file by invoking `read_index()`, which returns a list of `IndexEntry` objects representing each file staged in the index.

For each entry in the index:

- If `details` is `False`, print only the file path.
- If `details` is `True`:
  - Extract the stage number from the `flags` field by shifting right 12 bits and masking with `3` (binary `11`).
  - Format and print the file mode in octal, the SHA-1 hash in hexadecimal, the stage number, and the file path.

This allows users to quickly view the current contents of the index, optionally with rich detail useful for debugging or scripting.

#### Operation Steps

1. Call `read_index()` to parse the binary `.git/index` file and get a list of entries.
2. Iterate over each `IndexEntry`.
3. If `details` is `True`:
   - Calculate the stage number from entry flags.
   - Format and print the mode, SHA-1 hash, stage, and path.
4. Otherwise, print just the file path.

#### Example Usage

```python
# List just the file paths in the index
ls_files()

# List detailed information for each file in the index
ls_files(details=True)
```

Example output when `details=True`:

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 7b52009b64fd0a2a49e6d8a939753077792b0554 0    src/main.py
```

---

### Supporting Functions

The `ls_files` function relies heavily on the following supporting functions to operate correctly. These functions allow reading and interpreting the Git index and file data.

#### `read_index()`

Reads the `.git/index` file and returns a list of `IndexEntry` objects representing each tracked file.

- Validates the index file signature and version.
- Verifies the SHA-1 checksum of the index contents for integrity.
- Parses each index entry's fixed-length fields and variable-length path.
- Returns a fully populated list of entries.

This function is crucial for any operation that reads or manipulates the Git index.

---

### ASCII Diagram: Git Index File Structure

```
.git/index binary file layout:

+--------------------------+
| Signature: "DIRC" (4 B)  |
+--------------------------+
| Version (4 B)            |
+--------------------------+
| Number of Entries (4 B)  |
+--------------------------+
| Entry 1                  |
|   - Fixed fields (62 B)  |
|   - Path (variable)      |
|   - Padding              |
+--------------------------+
| Entry 2                  |
|   - Fixed fields (62 B)  |
|   - Path (variable)      |
|   - Padding              |
+--------------------------+
| ...                      |
+--------------------------+
| SHA-1 checksum (20 B)    |
+--------------------------+
```

Each `IndexEntry` contains metadata about a file staged in the index, such as timestamps, device and inode numbers, mode, UID/GID, file size, SHA-1 of content, flags, and the file path.

---

## Summary

The `ls_files` command is a straightforward utility to enumerate files staged in the Git index. By toggling the `details` flag, users can retrieve either a simple list of paths or a detailed view including modes, hashes, and stage numbers. This fits into the broader context of index management within the `pygit` project, enabling inspection and debugging of the index state before committing changes.

---

# Appendix: Full `ls_files` Function Code

```python
def ls_files(details=False):
    """Print list of files in index (including mode, SHA-1, and stage number
    if "details" is True).
    """
    for entry in read_index():
        if details:
            stage = (entry.flags >> 12) & 3
            print('{:6o} {} {:}\t{}'.format(
                    entry.mode, entry.sha1.hex(), stage, entry.path))
        else:
            print(entry.path)
```

---

# Cross References

- `read_index()` — Parses the Git index file. Documented in `read_index.md`.
- `read_file(path)` — Reads file contents as bytes. Documented in `read_file.md`.
- Other index management commands documented alongside in the **Index Management** section.