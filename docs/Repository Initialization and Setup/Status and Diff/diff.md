# diff.md - Displaying Diffs of Files Changed Between Index and Working Copy

---

## Overview

This document covers the functionality related to displaying differences (diffs) between files staged in the Git index and their corresponding versions in the working copy. It belongs to the **Status and Diff** section of the repository documentation, which focuses on inspecting and reporting the state of the working copy relative to the index and repository. The `diff.md` file details the mechanisms for detecting changed files and rendering line-by-line differences in a human-readable unified diff format.

The ability to display diffs is essential for users to review changes before committing, providing insight into what modifications will be recorded in the repository history. This module leverages core index reading, object retrieval, and file reading utilities to extract the relevant content and produce differences.

---

## Important Functions

### `diff()`

#### Purpose
The `diff()` function displays unified diffs for all files that have changes between the Git index and the working copy. It helps users visualize what edits have been made but not yet committed.

#### Parameters
- None

#### Operation
1. Calls `get_status()` to retrieve lists of changed, new, and deleted files; it focuses on `changed` files only.
2. Reads the Git index entries using `read_index()` and creates a dictionary keyed by file path.
3. For each changed file:
   - Retrieves the SHA-1 hash of the blob from the index entry.
   - Reads the blob object data from the Git object store with `read_object()`.
   - Decodes the blob content (file contents at the index state) into lines.
   - Reads the current working copy file content as lines.
   - Uses Python's `difflib.unified_diff` to generate a unified diff between the index and working copy versions.
   - Prints the diff line-by-line.
4. Prints a separator line (`'-' * 70`) between diffs of multiple files for clarity.

#### Example Usage
```python
# Show diffs for all changed files between index and working copy
diff()
```

---

### `get_status()`

#### Purpose
Returns the three categories of file paths indicating the status of the working copy relative to the index:
- Changed files (present in both but content differs)
- New files (present in working copy but not in index)
- Deleted files (present in index but missing in working copy)

#### Returns
Tuple of three lists: `(changed_paths, new_paths, deleted_paths)`

#### Operation
1. Walks the working copy directory tree (excluding `.git`) to list all files.
2. Reads the current index entries.
3. Computes:
   - Changed files: files present in both sets but whose content hashes differ.
   - New files: files present in working copy but not in index.
   - Deleted files: files present in index but not in working copy.
4. Returns sorted lists of these paths.

#### Example Usage
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `read_index()`

#### Purpose
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing staged files.

#### Returns
List of `IndexEntry` objects.

#### Operation
1. Reads the index file bytes.
2. Validates the SHA-1 checksum of the index data.
3. Parses the index header (signature, version, number of entries).
4. Iteratively unpacks each index entry including metadata and file path.
5. Returns all entries.

#### Example Usage
```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### `read_object(sha1_prefix)`

#### Purpose
Reads a Git object by its SHA-1 prefix from the object directory, decompresses, and returns its type and data.

#### Parameters
- `sha1_prefix`: SHA-1 hash prefix string identifying the object.

#### Returns
Tuple `(object_type, data_bytes)` where:
- `object_type` is a string such as `'blob'`, `'tree'`, or `'commit'`.
- `data_bytes` is the raw decompressed data of the object.

#### Operation
1. Uses `find_object()` to resolve the full object file path.
2. Reads and decompresses the stored object.
3. Parses the object header to identify type and content size.
4. Extracts and returns the object data.

#### Example Usage
```python
obj_type, data = read_object("a1b2c3d")
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### `read_file(path)`

#### Purpose
Reads the contents of a file from the working copy as bytes.

#### Parameters
- `path`: File path string.

#### Returns
Bytes content of the file.

#### Example Usage
```python
content = read_file("README.md")
print(content.decode())
```

---

## ASCII Diagram: Diff Display Workflow

```
+---------------------------+
| 1. Get status of changes  |
|    (get_status)           |
+------------+--------------+
             |
             v
+---------------------------+
| 2. Read index entries     |
|    (read_index)           |
+------------+--------------+
             |
             v
+---------------------------+
| 3. For each changed file: |
|    - Get SHA-1 from index |
|    - Read blob object     |
|    - Read working copy    |
|    - Generate unified diff|
+------------+--------------+
             |
             v
+---------------------------+
| 4. Print diffs to stdout  |
+---------------------------+
```

---

## Summary

The `diff.md` documentation outlines how diffs are displayed between the staged files in the Git index and the current working copy files. The process involves detecting changed files, retrieving their stored versions, reading current versions, and producing unified diffs for review. This functionality is critical for understanding what changes are about to be committed, supporting effective version control workflows.