# ls_files.md - Listing Files in the Git Index

## Overview

The `ls_files.md` document details the functionality for listing files tracked by Git's index within the repository. Specifically, it covers the `ls_files` function which reads the Git index and outputs a list of files currently staged for the next commit. This listing can be presented in a simple form (just file paths) or with detailed information including file mode, object SHA-1 hash, and stage number.

This file resides under the **Index Management** section in the overall documentation tree, highlighting its role in interacting with Git’s index data structures. It integrates closely with other index-related functions such as reading and writing the index (`read_index`, `write_index`), and relates to the diff and status functionalities by providing the foundational list of files tracked by Git.

---

## Function Documentation

### `ls_files(details=False)`

#### Purpose

Lists files currently staged in the Git index, optionally showing detailed information per entry. This is useful for inspecting what files Git is tracking at the index level before committing changes.

#### Parameters

- `details` (`bool`, optional): If `True`, prints detailed information for each file including:
  - File mode (permissions and type)
  - SHA-1 object hash of the file contents
  - Stage number (conflict stage in case of merges)
  
  If `False` (default), only the file paths are printed.

#### Operation

1. Calls `read_index()` to retrieve a list of `IndexEntry` objects representing each file staged in the index.
2. Iterates over each index entry.
3. If `details` is enabled:
   - Extracts the stage number from the flags attribute by right-shifting 12 bits and masking with `0x3`.
   - Prints the file mode in octal, the SHA-1 hash in hex, the stage number, and the file path.
4. Otherwise, prints just the file path.

#### Usage Example

```python
# List all files in the index by path only
ls_files()

# List files with detailed info: mode, SHA-1 hash, and stage number
ls_files(details=True)
```

#### Sample Output (when `details=True`)

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 3b18e2c4d4f7c0b3a1f2e1f0c5c3c7a4e2a4f9c0 0    src/main.py
```

---

## Related Functions and Concepts

To fully understand `ls_files`, it is helpful to know about:

- **`read_index()`**: Reads the `.git/index` file and returns a list of `IndexEntry` objects, each representing a tracked file with metadata such as timestamps, device/inode info, mode, SHA-1 hash, flags, and path.

- **Index Entry Structure:**

  Each entry in the index contains multiple fields packed in a binary format:

  ```
  +------------------+----------------+-------------------+
  | ctime (seconds)  | ctime (nanosec)| mtime (seconds)   |
  +------------------+----------------+-------------------+
  | mtime (nanosec)  | dev            | ino               |
  +------------------+----------------+-------------------+
  | mode             | uid            | gid               |
  +------------------+----------------+-------------------+
  | size             | SHA-1 (20 bytes)               | flags    |
  +---------------------------------------------------------+
  | path (variable length, null-terminated)                  |
  +---------------------------------------------------------+
  ```

- **Stage Number:**

  The stage number indicates the version of a file entry during a merge conflict (0 means no conflict). It is extracted from the upper bits of the flags field.

---

## ASCII Diagram: Index File Layout and `ls_files` Interaction

```
.git/
 ├── index    <-- Binary file storing index entries
 │
 └─> ls_files()
        │
        ├─> read_index()  -- Parses '.git/index' and returns list of IndexEntry
        │
        └─> For each IndexEntry:
              ├─ If details=True, print: mode, SHA-1, stage, path
              └─ Else, print: path
```

---

## Example Scenario

Suppose you have staged three files for your next Git commit. Running `ls_files()` will simply list their paths:

```
README.md
src/main.py
tests/test_main.py
```

Running `ls_files(details=True)` will provide extra insight into these files as stored in the index:

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 3b18e2c4d4f7c0b3a1f2e1f0c5c3c7a4e2a4f9c0 0    src/main.py
100644 7f3b2e3a5fa7c3e4f5a6b7c8d9e0f1a2b3c4d5e6 0    tests/test_main.py
```

This detailed output is valuable for debugging or scripting automation where you need exact object hashes or file modes.

---

## Summary

- The `ls_files` function provides a simple interface to list files tracked by Git’s index.
- Optional detailed output includes file mode, SHA-1 hash, and stage number.
- It depends on the `read_index` function to parse the Git index file.
- This listing is a crucial step before operations like `git commit`, diffing changes, or status reporting.

This documentation equips users and developers with the necessary understanding to leverage file listing from the Git index effectively.