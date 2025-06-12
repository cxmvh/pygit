# ls_files.md

## Overview

The `ls_files.md` file documents the functionality for listing files tracked in the Git index within the `pygit` project. It explains how to enumerate files present in the Git index with options to display detailed information such as file mode, SHA-1 object hash, and stage number. This documentation is part of the "Index and Working Copy Management" section, which covers managing the Git index and working copy files, including listing index entries, status checks, diffs, and staging. The `ls_files` functionality is a core utility for inspecting the current state of the index, which is crucial for operations like status reporting, committing, and staging changes.

---

## Function Documentation

### `ls_files(details=False)`

**Purpose:**  
List files currently recorded in the Git index. When `details` is `False` (default), only file paths are printed. When `details` is `True`, additional metadata about each file is shown, including the file mode, SHA-1 hash of the blob object, and the stage number.

**Parameters:**  
- `details` (`bool`): If `True`, output includes detailed information; otherwise, only file paths are printed.

**Operation:**  
1. Calls `read_index()` to read the Git index file and obtain a list of `IndexEntry` objects.  
2. Iterates over each index entry:  
   - If `details` is `True`:  
     - Extracts the stage number from the `flags` field using bit shifting and masking.  
     - Prints the file mode (in octal), the SHA-1 hash in hexadecimal, the stage number, and the file path.  
   - If `details` is `False`:  
     - Prints only the file path.

**Usage Example:**

```python
# List just the file paths in the index
ls_files()

# List files with detailed info (mode, SHA-1, stage)
ls_files(details=True)
```

**Sample Output (details=True):**

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100644 3b18e38b7a7c1e0c959ccf5f8f6a7f8d12345678 0    src/main.py
```

---

### Supporting Function: `read_index()`

**Purpose:**  
Read the Git index file (`.git/index`) and parse it into a list of `IndexEntry` objects representing tracked files and their metadata.

**Operation:**  
- Opens and reads the entire `.git/index` file as bytes.  
- Verifies the integrity of the index by checking the SHA-1 checksum at the end of the file.  
- Validates the index file signature (`'DIRC'`) and version number (currently 2).  
- Parses the binary entry records, each containing file metadata fields and the file path string.  
- Returns a list of `IndexEntry` objects.

**Usage Example:**

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.mode, entry.sha1.hex())
```

---

## Additional Context

The `ls_files` function depends on the `read_index()` function to obtain the current index state. The index entries provide the foundational data for numerous Git operations such as diffing, committing, and staging. Understanding the format and contents of the index entries is essential to grasp how Git tracks file versions internally.

---

## ASCII Diagram: Git Index Entry Fields and Output Mapping in `ls_files(details=True)`

```
+-----------------------------+        +-------------------------------+
|        IndexEntry           |        |       Output Fields           |
+-----------------------------+        +-------------------------------+
| mode            (int)       |  --->  | file mode (octal)              |
| sha1            (bytes)     |  --->  | SHA-1 hash (hex string)        |
| flags           (int)       |  --->  | stage number (bits 12-13)      |
| path            (string)    |  --->  | file path                     |
+-----------------------------+        +-------------------------------+

Stage number extraction:
flags: [ ... | bits 13 | bit 12 | ... ]
           \___________/
               & 3 (mask to get 2 bits)
```

---

## Summary

The `ls_files` function is a straightforward yet vital utility in the `pygit` project that exposes the contents of the Git index, optionally including detailed metadata. It serves as a user-facing command to inspect which files are staged or tracked and plays a foundational role in index and working copy management workflows.