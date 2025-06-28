# ls_files.md

## Overview

The `ls_files` module provides functionality to list the files currently staged in the Git index. It is part of the **Index Management** subsection within the **Working Copy Status and Diff** section of the pygit documentation. This module enables users and tools to inspect the contents of the index, which serves as the staging area for files before committing.

Listing files in the index is fundamental for understanding the current tracked state of the repository and for verifying what is staged for the next commit. This module supports both simple listing of file paths and detailed listing including file mode, SHA-1 object hash, and stage number.

---

## Functions

### `ls_files(details=False)`

#### Purpose

Prints a list of files that are currently tracked in the Git index. When the `details` parameter is set to `False` (default), it outputs only the file paths. If `details` is `True`, it prints additional information for each file including:

- File mode (in octal format)
- SHA-1 hash of the blob object representing the file content
- Stage number (used internally by Git to represent merge conflicts)
- File path

This function reads the index entries and formats output accordingly.

#### Parameters

- `details` (bool): Whether to include detailed information about each file. Default is `False`.

#### How It Works

1. Calls the `read_index()` function to retrieve the list of `IndexEntry` objects from the Git index file.
2. Iterates over each entry.
3. If `details` is `True`:
   - Extracts the stage number from the flags field by right-shifting 12 bits and masking with 3 (`(entry.flags >> 12) & 3`).
   - Prints the mode, SHA-1 (hexadecimal), stage, and path formatted as:
     ```
     <mode> <sha1> <stage>  <path>
     ```
4. If `details` is `False`:
   - Prints only the file path.

#### Usage Example

```python
import pygit

# List all files in the index (paths only)
pygit.ls_files()

# Output might be:
# README.md
# src/main.py
# tests/test_main.py

# List all files with detailed info
pygit.ls_files(details=True)

# Output might be:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0	README.md
# 100755 3a4f8d6c9d7e5b1a2c3d4e5f6071829a0b1c2d3e 0	src/main.py
# 100644 8b1a9953c4611296a827abf8c47804d7f9a1e0e1 0	tests/test_main.py
```

---

## Supporting Functions (for context)

### `read_index()`

Reads the `.git/index` file and returns a list of `IndexEntry` objects representing the current state of the index.

- Validates the index file signature and version.
- Checks SHA-1 checksum for integrity.
- Iteratively unpacks each index entry, including metadata and file path.

### Data Flow Diagram

```
+-----------------+
|  pygit.ls_files  |
+-----------------+
         |
         v
+-----------------+
|  read_index()    | -- reads --> .git/index file
+-----------------+
         |
         v
+-----------------+
|  IndexEntry[]   | -- iteration over entries
+-----------------+
         |
         v
+----------------------------+
|  Prints file info to stdout |
+----------------------------+
```

---

## Notes

- The `stage` number is relevant when handling merge conflicts (multiple stages for a single path). In normal cases, this is 0.
- File mode is shown in octal notation (e.g., `100644` for a regular file).
- The SHA-1 hash corresponds to the blob object in the Git object database that stores the file content as staged.

---

This module is typically used by commands that report on the index status or allow users to inspect what is staged. It interacts closely with `read_index()` and related index and object handling functions documented elsewhere in the pygit project.