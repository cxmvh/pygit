# ls_files.md

# Listing Files in the Git Index (`ls_files`)

## Overview

This document details the functionality for listing files tracked in the Git index within the pygit project. It focuses on the `ls_files` operation, which provides a way to view files currently staged in the index, optionally with detailed metadata such as file mode, SHA-1 hash, and stage number. This feature is part of the **Index Management** section of the documentation tree, which covers operations related to reading, writing, and manipulating the Git index file and its entries.

Understanding and utilizing `ls_files` is critical for developers and users who want to inspect the state of the index — the staging area where file changes are prepared before committing — and to gain insights into the metadata associated with each tracked file. This complements other index-related operations such as adding files, writing entries, and checking status.

---

## Function Documentation

### `ls_files(details=False)`

#### Purpose

The `ls_files` function lists files currently recorded in the Git index. It can display just the file paths or, optionally, detailed information about each file entry including:

- File mode (permissions and type),
- SHA-1 hash of the file content,
- Stage number (indicating merge conflict stages).

This is analogous to the `git ls-files` command in Git and is useful for inspecting the precise contents and metadata of the staging area.

#### Parameters

- `details` (bool, optional):  
  When set to `False` (default), only the file paths are printed.  
  When set to `True`, detailed information is printed for each file.

#### Operation Steps

1. Calls the helper function `read_index()` to read and parse the Git index file (`.git/index`), returning a list of `IndexEntry` objects.
2. Iterates over each `IndexEntry`:
   - If `details` is `False`, prints the file path directly.
   - If `details` is `True`:
     - Extracts the stage number by bit-shifting and masking the `flags` attribute (`(entry.flags >> 12) & 3`).
     - Prints the mode in octal format, SHA-1 hash as a hexadecimal string, stage number, and file path in a formatted line.
3. Outputs are printed to standard output.

#### Example Usage

```python
# List only file paths currently tracked in the index
ls_files()

# Output:
# README.md
# src/main.py
# docs/usage.md

# List files with detailed info: mode, SHA-1 hash, and stage
ls_files(details=True)

# Example output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
# 100755 4b825dc642cb6eb9a060e54bf8d69288fbee4904 0    src/main.py
# 100644 7f6a8f799f13d3a7c1f7f3c2e4b620f7aa3c9385 0    docs/usage.md
```

---

### Supporting Function: `read_index()`

#### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing each file entry in the index.

#### Operation Highlights

- Reads raw binary data from `.git/index`.
- Validates the index signature (`b'DIRC'`) and version (2).
- Verifies the SHA-1 checksum at the end of the index file.
- Parses each index entry by unpacking fixed-length fields and reading the variable-length file path.
- Returns a list of entries with attributes such as `mode`, `sha1`, `flags`, and `path`.

#### Example Call

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

## ASCII Diagram: Git Index Entry Structure and `ls_files` Flow

```
+---------------------------------------------------------+
|                     .git/index file                     |
|                                                         |
|  +---------+---------+---------+---------+-------------+|
|  | Header  | Entry 1 | Entry 2 |  ...    |  Checksum   ||
|  | (Signature,Version,EntryCount)                       ||
|  +---------+---------+---------+---------+-------------+|
|                                                         |
+---------------------------------------------------------+

Each Entry:

+--------------------------+
| ctime_s (4 bytes)         |
| ctime_n (4 bytes)         |
| mtime_s (4 bytes)         |
| mtime_n (4 bytes)         |
| dev (4 bytes)             |
| ino (4 bytes)             |
| mode (4 bytes)            |
| uid (4 bytes)             |
| gid (4 bytes)             |
| size (4 bytes)            |
| sha1 (20 bytes)           |
| flags (2 bytes)           |
| path (variable bytes)     |
| padding (to 8-byte align) |
+--------------------------+

`ls_files` flow:

read_index()
     |
     +--> list of IndexEntry
             |
             +--> for each entry:
                     if details:
                         print(mode, sha1, stage, path)
                     else:
                         print(path)
```

---

## Related Documentation

- See [index.md](index.md) for details on the Git index file format and `IndexEntry` structure.
- See [status.md](../Working Copy Status and Diff/status.md) for commands that use the index to report working directory status.
- See [diff.md](../Working Copy Status and Diff/diff.md) for generating diffs between the index and working directory.

---

## Summary

The `ls_files` function is a straightforward yet essential tool for listing the contents of the Git index. By optionally providing detailed information, it enables users to inspect the staging area in depth, which is crucial for understanding the state of files before committing changes. This documentation, alongside related index management files, helps developers leverage and extend the Git index functionality within the pygit project.