# status.md

# Status Reporting Functions Documentation

---

## Overview

This document provides comprehensive technical documentation on the status reporting functions implemented in the `pygit` repository, specifically focusing on `get_status` and `status`. These functions are central to tracking the state of the working copy and the Git index, identifying changes such as file modifications, additions, and deletions, and displaying diffs between the index and working directory files.

Situated within the **Working Copy and Index Management** section of the broader documentation tree, this file details how `pygit` monitors file states to provide users with insights into repository changes before committing. It integrates closely with other components like `read_index`, `hash_object`, and `diff` to offer a robust status reporting mechanism.

---

## Function Documentation

---

### `get_status()`

#### Purpose

`get_status` analyzes the current working directory and Git index to determine which files have changed, which are new (untracked), and which have been deleted. It returns a tuple containing three sorted lists:

- `changed_paths`: Files modified since the last indexing.
- `new_paths`: Files present in the working directory but not in the index.
- `deleted_paths`: Files present in the index but missing in the working directory.

This function is a foundational utility for status reporting, enabling higher-level commands to display repository state.

#### Parameters

None.

#### Returns

```python
(changed_paths: List[str], new_paths: List[str], deleted_paths: List[str])
```

Each list contains file paths relative to the repository root.

#### Operation Steps

1. Traverse the working directory recursively, excluding the `.git` directory.
2. Normalize file paths to POSIX-style (with forward slashes).
3. Collect all current file paths in the working directory into a set.
4. Read the Git index entries and map them by file path.
5. Determine:
   - **Changed files:** Intersection of working directory and index paths where the file content hash (`blob` SHA-1) differs.
   - **New files:** Files present in the working directory but not in the index.
   - **Deleted files:** Files present in the index but missing in the working directory.
6. Return the three categories as sorted lists.

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

The `status` function provides a user-friendly display of the current repository status by printing lists of changed, new, and deleted files. It internally uses `get_status` to gather the file states.

#### Parameters

None.

#### Operation Steps

1. Call `get_status` to retrieve lists of changed, new, and deleted files.
2. For each non-empty list, print an appropriate header followed by indented file paths.
3. If no files exist in a category, that category is not printed.

#### Example Usage

```python
status()
# Output:
# changed files:
#     file1.py
#     src/module2.c
# new files:
#     README.md
# deleted files:
#     old_script.sh
```

---

### `diff()`

#### Purpose

`diff` shows the differences between the Git index and the working copy for all changed files. It is a detailed complement to `status` and `get_status`, helping users inspect exactly what changes have occurred.

#### Parameters

None.

#### Operation Steps

1. Call `get_status` to retrieve the list of changed files.
2. Read the index entries into a dictionary keyed by file path.
3. For each changed file:
   - Retrieve the stored blob object from the index.
   - Read the current working copy file contents.
   - Use Python's `difflib.unified_diff` to produce a unified diff between the index and working copy.
4. Print the diff output to stdout.
5. Separate diffs for multiple files with a line of dashes.

#### Example Usage

```python
diff()
# Output:
# --- file1.py (index)
# +++ file1.py (working copy)
# @@ -1,4 +1,5 @@
#  ...
# ----------------------------------------------------------------------
# --- src/module2.c (index)
# +++ src/module2.c (working copy)
# @@ -20,7 +20,8 @@
#  ...
```

---

### `ls_files(details=False)`

#### Purpose

`ls_files` lists files recorded in the Git index. When `details` is `True`, it prints additional metadata such as file mode, SHA-1 hash, and stage number.

#### Parameters

- `details` (bool): If `True`, detailed info is printed; otherwise, only file paths are listed.

#### Operation Steps

1. Read the index entries.
2. For each entry:
   - If `details` is `True`, extract and print mode, SHA-1 hash, stage number, and path.
   - Else, print only the file path.

#### Example Usage

```python
# Simple list of files
ls_files()

# Detailed listing
ls_files(details=True)
# Output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
# 100644 3a7bd3e2360a7a7a0a6eab7c6f4e0b7bee3f5cd9 0    src/main.c
```

---

### Supporting Functions (Referenced by Status Functions)

---

#### `read_index()`

Reads and parses the Git index file, returning a list of `IndexEntry` objects representing files staged for commit.

---

#### `read_file(path)`

Reads and returns the byte content of the specified file path.

---

#### `hash_object(data, obj_type, write=True)`

Computes the SHA-1 hash of the given data as a Git object of the specified type (usually `'blob'` for files). Optionally writes the object to the Git object store.

---

#### `read_object(sha1_prefix)`

Reads a Git object by SHA-1 prefix and returns its type and data.

---

## ASCII Diagram: How Status Functions Interact with Repository Components

```
+------------------+
|  Working Copy    |  <-- Files on disk
+---------+--------+
          |
          | read_file()
          v
+------------------+         +-------------------+
|   get_status()   |<-------> |    read_index()    |  <-- Index entries
+---------+--------+         +-------------------+
          |
          | hash_object() for each file
          v
+-------------------------+
| Compare Working Copy and |
|     Index SHA-1 hashes  |
+------------+------------+
             |
             | returns lists of changed, new, deleted files
             v
+------------------+
|     status()     |  <-- Prints summary to user
+------------------+

+------------------+
|      diff()      |  <-- Shows detailed diffs (index vs working copy)
+------------------+
```

---

## Summary

The status reporting functions in `pygit` provide essential insights into the repository state, bridging the gap between the working directory and the Git index. By detecting changes, additions, and deletions, and facilitating diff views, these functions empower users to understand and manage changes effectively before committing.

This documentation covers the main functions `get_status()`, `status()`, and `diff()` with usage examples and explanations, situating them within the broader repository management workflows.