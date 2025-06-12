# ls_files.md

# Documentation for Listing Files in the Git Index

---

## Overview

The `ls_files.md` file documents functionality related to listing files present in the Git index (also known as the staging area). The Git index is a critical component in Git's architecture, representing a snapshot of the working directory that will be committed in the next commit. This document explains how to list files tracked in the index, optionally showing detailed metadata such as file mode, SHA-1 object hashes, and staging information. It fits within the broader "Index Management" section of the project documentation, which covers reading, writing, and manipulating the Git index. The functionality provided here is essential for understanding the state of files staged for commit and for debugging or inspecting the index contents.

---

## Function Documentation

### Function: `ls_files(details=False)`

#### Purpose

The `ls_files` function lists files currently recorded in the Git index. It can output either a simple list of file paths or, if detailed information is requested, it includes the file mode (permissions), the SHA-1 object hash of the file's blob, and the stage number. This mirrors the behavior of the `git ls-files` command in Git.

#### Parameters

- `details` (bool, optional):  
  If `False` (default), the function prints only the file paths.  
  If `True`, the function prints detailed information for each index entry.

#### Preconditions

- The Git index file must exist and be valid in the `.git/index` path.
- The index file is expected to be version 2 and checksum-verified.

#### Operation Details

1. Calls `read_index()` to read and parse the Git index file, returning a list of `IndexEntry` objects representing files in the index.
2. Iterates over each index entry.
3. If `details` is `True`:
   - Extracts the stage number from `entry.flags` (bits 12-13).
   - Prints the file mode in octal, SHA-1 hash in hexadecimal, stage number, and the file path, all formatted neatly.
4. If `details` is `False`:
   - Prints only the file path.

#### Example Usage

```python
# Basic usage: print all file paths in the index
ls_files()

# Detailed usage: print mode, SHA-1, stage, and file path
ls_files(details=True)
```

#### Sample Output (details=True)

```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
100755 1f8ac10f23c5b5bc1167bda84b833e5c057a77d2 0    script.sh
```

---

### Supporting Function: `read_index()`

#### Purpose

Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects. This function performs validation of the index file signature, version, and checksum to ensure integrity.

#### Operation Summary

- Opens `.git/index` and reads its binary data.
- Validates the checksum at the end of the index file.
- Parses the header to confirm the signature "DIRC" and version (expected 2).
- Iterates through the entries, unpacking fixed-size fields plus variable-length file path strings.
- Decodes file paths and constructs `IndexEntry` objects.
- Returns a list of all index entries.

#### Example Usage

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

## ASCII Diagram: Relationship of Index Entries and the `ls_files` Output

```
+-----------------+            +-----------------------------+
|  Git Index File  |  read_index()  | List of IndexEntry Objects  |
+-----------------+  ---------> +-----------------------------+
                                      |
                                      | Iteration over entries
                                      v
                             +-------------------------+
                             |    ls_files(details)    |
                             +-------------------------+
                                      |
                +---------------------+---------------------+
                |                                           |
          details=False                                details=True
                |                                           |
      Print file paths only                   Print mode, SHA-1, stage, and path
                |                                           |
        e.g., README.md                          e.g., 100644 e69d... 0 README.md
```

---

## Additional Context

The `ls_files` function is commonly used to inspect the current staging area and is integrated with other index management operations such as adding files (`add()`), writing the index (`write_index()`), and creating commit trees (`write_tree()`). It helps users or scripts verify what is staged before committing changes.

---

# Summary

This documentation covers the `ls_files` function, detailing how to list files recorded in the Git index in both simple and detailed forms. It relies on the foundational `read_index` function to parse the Git index file. The functionality is crucial for interacting with the Git staging area and forms part of the broader index management toolkit.