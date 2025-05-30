# diff.md

# Documentation for diff functions showing changes between index and working copy files.

---

## Overview

The `diff.md` file documents the functionality related to showing differences between files stored in the Git index and their corresponding versions in the working copy. This file is part of the "Working Copy and Index Management" section of the repository documentation. It details how the diff operation works within the `pygit` project, illustrating how changes in tracked files are detected and presented to the user. The diff functionality is crucial to understanding changes before committing, providing a line-by-line comparison to assist developers in reviewing modifications.

This document covers the primary `diff()` function and explains its interaction with supporting functions such as `get_status()`, `read_index()`, `read_object()`, and `read_file()`. Additionally, it includes usage examples and explanations to facilitate integrating and extending diff-related features.

---

## Function Documentation

### `diff()`

**Purpose:**  
Display the differences between the files recorded in the Git index and their corresponding versions in the working copy. This function identifies files that have changed and outputs unified diffs to standard output, showing line-by-line modifications.

**Parameters:**  
- None

**Operation Steps:**  
1. Call `get_status()` to obtain a list of changed files by comparing the working copy with the index.
2. Read the current index entries using `read_index()` and create a mapping from file paths to index entries.
3. Iterate over each changed file path:
   - Retrieve the blob SHA-1 hash from the index entry.
   - Use `read_object()` to read the blob object from the Git object store.
   - Decode the blob data into lines representing the file content in the index.
   - Read the current file contents from the working copy using `read_file()` and decode into lines.
   - Use Python's `difflib.unified_diff()` to generate a unified diff between the index version and the working copy.
   - Print each line of the diff output.
4. Separate multiple diffs with a line of dashes (`----------------------------------------------------------------------`) for clarity.

**Example Usage:**

```python
# Show diffs of all changed files between index and working copy
diff()
```

**Output Example:**

```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
 Line 1
-Line 2
+Modified Line 2
 Line 3
+New Line 4
----------------------------------------------------------------------
```

---

### `get_status()`

**Purpose:**  
Determine the status of the working copy relative to the Git index, returning tuples of changed, new, and deleted file paths.

**Returns:**  
- Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`

**Operation Steps:**  
1. Walk the working directory tree, excluding `.git` directories, to collect all file paths.
2. Read the index entries and map them by path.
3. Compute:
   - `changed`: Files present in both working copy and index with differing contents.
   - `new`: Files present in working copy but not in index.
   - `deleted`: Files present in index but missing from working copy.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `read_index()`

**Purpose:**  
Read the Git index file and parse its contents into a list of `IndexEntry` objects representing tracked files.

**Returns:**  
- List of `IndexEntry` objects.

**Operation Steps:**  
1. Read raw bytes from `.git/index`.
2. Validate index file signature and version.
3. Extract entries by unpacking fixed-size metadata and associated file path strings.
4. Verify checksum for data integrity.

**Example Usage:**

```python
index_entries = read_index()
for entry in index_entries:
    print(f"{entry.path} - SHA1: {entry.sha1.hex()}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Read a Git object (blob, tree, commit) identified by its SHA-1 prefix and return its type and raw data.

**Parameters:**  
- `sha1_prefix` (str): Prefix of the SHA-1 hash identifying the object.

**Returns:**  
- Tuple `(object_type, data_bytes)`

**Operation Steps:**  
1. Locate object file by SHA-1 prefix.
2. Decompress stored zlib-compressed object data.
3. Parse header to extract object type and size.
4. Return type and raw data bytes.

**Example Usage:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file in the working copy as bytes.

**Parameters:**  
- `path` (str): Path to the file.

**Returns:**  
- Bytes of the file contents.

**Example Usage:**

```python
content_bytes = read_file('example.txt')
print(content_bytes.decode())
```

---

## ASCII Diagram: Diff Workflow

```
+------------------+           +------------------+           +------------------+
|  Working Copy    |           |      Index       |           |   Git Object     |
|  (filesystem)    |           |  (tracked files) |           |    Store         |
+--------+---------+           +---------+--------+           +---------+--------+
         |                               |                              |
         | read_file(path)               | read_index()                | read_object(sha1)
         |                               |                              |
         v                               v                              v
+------------------+           +------------------+           +------------------+
| Current file     |           | Index entry for  |           | Blob object data  |
| contents         |           | each tracked file|           | (file snapshot)   |
+--------+---------+           +---------+--------+           +---------+--------+
         \                       /                                   /
          \                     /                                   /
           \                   /                                   /
            +-----------------+----------------------------------+
                              |
                              | difflib.unified_diff()
                              |
                              v
                    +----------------------+
                    | Unified diff output   |
                    +----------------------+
```

---

# Summary

This documentation outlines how the `diff()` function and its helper functions work together to show changes between the Git index and working copy files. Understanding these functions enables developers to inspect modifications before committing, ensuring code integrity and facilitating version control workflows.