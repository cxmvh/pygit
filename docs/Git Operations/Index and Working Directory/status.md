# status.md

# Displaying the Status of the Working Copy

## Overview

The `status.md` file documents the functionality for inspecting and displaying the status of the working copy in the `pygit` project. It is part of the **Working Copy Status and Diff** section within the documentation tree, which provides tools for examining the differences between the working directory and the Git index. This file focuses on reporting which files have changed, which are new, and which have been deleted, facilitating users to understand and manage their current working state before committing or pushing changes. The key functions described here are `status()` and `get_status()`, supported by utility functions like `read_index()`, `read_file()`, and `hash_object()`.

---

## Function Documentation

### `get_status()`

#### Purpose
Determines the status of the working copy by comparing the files present in the working directory with those recorded in the Git index. It returns three lists categorizing files as changed, new, or deleted.

#### Parameters
None

#### Returns
A tuple of three lists:
- `changed_paths`: Files present both in the working directory and index but with differing contents.
- `new_paths`: Files present in the working directory but not yet added to the index.
- `deleted_paths`: Files present in the index but missing from the working directory.

#### Operation Details
1. Traverse the current directory recursively (excluding `.git`), collecting all file paths.
2. Read the Git index entries and map them by their file paths.
3. Determine:
   - Changed files: Intersection of working directory and index files where the content hash differs.
   - New files: Files in the working directory but not in the index.
   - Deleted files: Files in the index but missing from the working directory.
4. Return sorted lists of these categories.

#### Example Usage
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `status()`

#### Purpose
Displays a human-readable summary of the working copy status by printing lists of changed, new, and deleted files.

#### Parameters
None

#### Operation Details
1. Calls `get_status()` to obtain categorized file lists.
2. Prints headers and file paths for each category if they are not empty.

#### Example Usage
```python
status()
# Possible Output:
# changed files:
#     src/main.py
# new files:
#     docs/readme.md
# deleted files:
#     old_script.py
```

---

### Supporting Functions

#### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects representing files currently tracked in the index. 

This function is critical for mapping file paths to their stored SHA-1 hashes to detect changes.

---

#### `read_file(path)`

Reads and returns the content of the file at the specified `path` as bytes. Used to obtain current file contents for hashing.

---

#### `hash_object(data, obj_type, write=False)`

Computes the SHA-1 hash of the given data with the specified Git object type (usually `'blob'` for file contents). When `write` is `False`, it only computes the hash without storing the object.

Used by `get_status()` to compare current file contents against the indexed version.

---

## ASCII Diagram: Working Copy Status Overview

```
+-------------------+          +------------------+
| Working Directory  |          |  Git Index       |
| (Current files)    |          | (Tracked files)  |
+-------------------+          +------------------+
          |                             |
          |                             |
          |       Compare file hashes  |
          |---------------------------->
          |                             |
 +-------------------+         +-------------------+
 | Changed files     |         | New files          |
 | (in both but diff)|         | (only in working   |
 +-------------------+         |  directory)        |
                               +-------------------+
          |                             |
          |                             |
          |<----------------------------|
          |                             |
 +-------------------+                 |
 | Deleted files     |                 |
 | (only in index)   |                 |
 +-------------------+                 |
```

---

## Summary

The status functionality in `status.md` is foundational for any Git workflow, enabling users to inspect which files have been altered, added, or removed before staging, committing, or pushing. By leveraging file system traversal, index reading, and Git object hashing, it provides an accurate and efficient snapshot of the working copy's state. The `status()` command offers a straightforward way to visualize this information in the console.