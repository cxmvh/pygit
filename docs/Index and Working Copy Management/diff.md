# diff.md - Showing Diffs Between the Index and Working Copy Files

---

## Overview

The `diff.md` file documents the functionality for displaying differences between files in the Git index and the working copy. This is a key feature in the "Status and Diff" section of the overall project documentation. It enables users to see line-by-line changes in files that have been modified but not yet staged, facilitating informed decisions before commits. The functionality primarily revolves around reading the Git index, comparing file contents with the working directory, and presenting unified diffs in a human-readable format.

---

## Function Documentation

### `diff()`

**Purpose:**  
Shows the line-by-line differences between files recorded in the Git index and their counterparts in the working copy. It lists all changed files and prints a unified diff for each, highlighting additions, deletions, and modifications.

**Parameters:**  
None

**Operation Steps:**  
1. Retrieve the list of changed file paths using `get_status()`. This returns files that exist both in the index and working copy but whose contents differ.  
2. Read all entries from the Git index using `read_index()` and map them by their file path for quick lookup.  
3. For each changed file path:  
    - Fetch the SHA-1 hash of the file blob stored in the index.  
    - Read the blob object data (file content) from the Git object store using `read_object(sha1)`.  
    - Read the current file content from the working copy using `read_file(path)`.  
    - Decode both contents into lines and generate a unified diff using `difflib.unified_diff()`.  
    - Print the diff output to stdout.  
    - Print a separator line of dashes (`-`) between diffs for readability, except after the last file.

**Example Usage:**  
```python
diff()
```

**Sample Output:**  
```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,4 +1,5 @@
 Line one
-Line two
+Line two modified
 Line three
+New line added
----------------------------------------------------------------------
```

---

### Supporting Functions Used by `diff()`

#### `get_status()`

Returns three lists: changed files, new files, and deleted files by comparing the working copy and the index.

```python
changed, new, deleted = get_status()
```

#### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects representing files tracked by Git.

```python
entries = read_index()
entries_by_path = {entry.path: entry for entry in entries}
```

#### `read_object(sha1_prefix)`

Reads a Git object by its SHA-1 prefix and returns a tuple `(object_type, data_bytes)`.

```python
obj_type, data = read_object(sha1)
```

#### `read_file(path)`

Reads the contents of a file from the working copy as bytes.

```python
content_bytes = read_file(path)
```

---

## ASCII Diagram: Diff Workflow

```
+------------------+           +-----------------+           +-------------------+
|  Git Index       |           | Working Copy    |           | Unified Diff      |
|  (Stored blobs)  |           | (Current files) |           | (Console Output)  |
+---------+--------+           +--------+--------+           +---------+---------+
          |                             |                              ^
          |                             |                              |
          |                             |                              |
          | 1. Identify changed files   |                              |
          |---------------------------->|                              |
          |                             |                              |
          | 2. Read blob data (index)  |                              |
          |<----------------------------                              |
          |                             |                              |
          | 3. Read file contents       |                              |
          |                             |----------------------------->|
          |                             |                              |
          | 4. Generate unified diff    |                              |
          |                             |                              |
          | 5. Print diff output        |                              |
          +-----------------------------                              |
                                                                       |
         ---------------------------------------------------------------
```

---

## Summary

The `diff()` function is an essential utility for inspecting differences between the staged (indexed) version of files and the current working copy. By leveraging reading of index entries and Git objects, and utilizing Python's `difflib` module, it produces a standard unified diff output familiar to Git users. This facilitates transparent change tracking before committing, improving workflow accuracy and awareness.

---

# End of diff.md Documentation