# diff.md

# Documentation for `diff` function displaying differences between index and working copy

---

## Overview

The `diff.md` document provides detailed documentation for the `diff()` function within the `pygit` project. This function is responsible for displaying line-by-line differences between the files currently staged in the Git index and the files in the working copy (filesystem). It is a key utility in the "Index and Working Copy Management" section of the documentation, illustrating how changes in files are detected and presented to the user before committing.

This file fits within the broader context of managing the Git index and working copy, complementing other functions such as `add()`, `status()`, and `read_index()`. It leverages core object handling functions such as `read_object()` and file reading utilities to access and compare file contents.

---

## Function Documentation

### `diff()`

#### Purpose

The `diff()` function displays the differences between the versions of files stored in the Git index and the corresponding files in the working copy. It helps users understand what changes have been made to files that are staged for commit versus what is currently on disk, by showing a unified diff output for each changed file.

#### Parameters

- This function takes no parameters.

#### Preconditions

- The Git index file (`.git/index`) must be readable and correctly formatted.
- The working copy files must exist at the paths listed in the index.
- Related functions like `get_status()`, `read_index()`, `read_object()`, and `read_file()` must be available and working correctly.

#### Operation Details

1. Retrieves the list of changed files by calling `get_status()`, which compares the working copy and index.
2. Reads the current Git index entries using `read_index()`, constructing a dictionary mapping file paths to their corresponding index entries.
3. Iterates over each changed file path:
   - Extracts the SHA-1 hash of the file blob stored in the index.
   - Reads the blob object from the object store using `read_object()`.
   - Decodes the blob data (file content as bytes) into a list of lines.
   - Reads the current version of the file from the working copy and splits it into lines.
   - Uses Python's `difflib.unified_diff()` to generate a unified diff between the index version and the working copy version of the file.
   - Prints the diff output to stdout.
4. Prints a separator line (`----------------------------------------------------------------------`) between diffs of multiple files for readability.

#### Example Usage

```python
>>> diff()
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
 Line 1 of file
-Line 2 of file
+Modified Line 2 of file
 Line 3 of file
+Added Line 4 of file
----------------------------------------------------------------------

--- src/main.py (index)
+++ src/main.py (working copy)
@@ -10,7 +10,8 @@
-def old_function():
-    pass
+def new_function():
+    print("Updated function")
```

#### Explanation Diagram

```
+------------------+      +-------------------+      +--------------------+
| Git Index        |      | Working Copy      |      | diff() Function     |
| (.git/index)     |      | (Filesystem)      |      |                    |
| - Stores blobs   |      | - Current files   |      | - Reads index      |
|   with SHA-1     |      | - Possibly changed|      | - Reads blobs from | 
+--------+---------+      +---------+---------+      |   object store     |
         |                          |                | - Reads working    |
         |                          |                |   copy files       |
         |                          |                | - Compares content |
         |                          |                | - Prints unified   |
         |                          |                |   diff output      |
         +--------------------------+---------------->+--------------------+
```

---

### Related Functions

For a full understanding of `diff()`, it is helpful to be familiar with the following related functions:

- **`get_status()`**: Determines which files are changed, new, or deleted by comparing the index and working copy.
- **`read_index()`**: Reads the Git index file and returns entries representing staged files.
- **`read_object(sha1_prefix)`**: Reads and decompresses Git objects from the object store based on SHA-1 hash prefixes.
- **`read_file(path)`**: Reads a file from the working copy as bytes.

---

## Supporting Function Summaries (Key for `diff()`)

### `get_status()`

Returns three lists of file paths: changed, new, and deleted files by comparing the working directory and index.

```python
changed, new, deleted = get_status()
```

### `read_index()`

Reads the `.git/index` file and returns a list of `IndexEntry` objects.

```python
entries = read_index()
```

### `read_object(sha1_prefix)`

Reads a Git object by SHA-1 prefix and returns a tuple `(obj_type, data_bytes)`.

```python
obj_type, data = read_object(sha1)
```

### `read_file(path)`

Reads the file at `path` and returns its contents as bytes.

```python
content_bytes = read_file(path)
```

---

## Detailed Code Snippet of `diff()`

```python
def diff():
    """Show diff of files changed (between index and working copy)."""
    changed, _, _ = get_status()
    entries_by_path = {e.path: e for e in read_index()}
    for i, path in enumerate(changed):
        sha1 = entries_by_path[path].sha1.hex()
        obj_type, data = read_object(sha1)
        assert obj_type == 'blob'
        index_lines = data.decode().splitlines()
        working_lines = read_file(path).decode().splitlines()
        diff_lines = difflib.unified_diff(
                index_lines, working_lines,
                '{} (index)'.format(path),
                '{} (working copy)'.format(path),
                lineterm='')
        for line in diff_lines:
            print(line)
        if i < len(changed) - 1:
            print('-' * 70)
```

---

## Summary

The `diff()` function is a critical component in the pygit implementation that allows users to inspect changes before committing. By leveraging the Git index and working copy comparison, it provides a clear, line-by-line view of modifications, enabling informed decisions during version control workflows. This documentation covers its purpose, usage, and interaction with other core pygit functions.