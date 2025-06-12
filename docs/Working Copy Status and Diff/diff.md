# diff.md

## Overview

The `diff.md` file documents the functionality for displaying differences ("diffs") between the Git index (staged files) and the working copy (current files in the filesystem). This feature is a crucial part of the "Working Copy Status and Diff" section of the overall documentation, which focuses on providing utilities to inspect and manage changes in the working directory relative to the index.

The ability to show diffs enables users and internal Git operations to understand exactly what changes have been made but not yet staged or committed. This documentation covers the key function `diff()`, which leverages other components such as reading the index, fetching object contents, reading working files, and computing textual diffs.

---

## Functions

### `diff()`

#### Purpose

Show a unified diff of all files that have been changed between the Git index and the working copy. This allows users to see line-by-line changes that exist in their working directory compared to what is currently staged in the index.

#### Parameters

- None.

#### Preconditions

- The Git index file must be present and readable.
- The changed files must exist in both the index and the working directory.
- Git objects for the indexed files’ blobs must be present and accessible.

#### Operation Steps

1. Use `get_status()` to obtain lists of changed, new, and deleted files. The `diff()` function only considers the changed files.
2. Read all entries from the Git index using `read_index()`, and create a dictionary mapping file paths to their corresponding index entries.
3. For each changed file path:
   - Retrieve the SHA-1 hash of the blob object stored in the index.
   - Read the blob object data from the object store using `read_object()`. This gives the file content as stored in the index.
   - Read the current content of the file in the working directory using `read_file()`.
   - Decode both contents to text lines.
   - Use Python's `difflib.unified_diff()` to generate a unified diff between the indexed content and the working copy content.
   - Print the diff output line-by-line.
   - If there are multiple changed files, print a separator line (`----------------------------------------------------------------------`) between diffs.

#### Example Usage

```python
# Show diffs for all changed files between index and working copy
diff()
```

##### Sample Output Snippet

```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
-Line 1 of file
+Line 1 of file (modified)
 Line 2 of file
+New line 3 added
----------------------------------------------------------------------
--- another_file.py (index)
+++ another_file.py (working copy)
@@ -10,7 +10,8 @@
-def foo():
-    pass
+def foo():
+    print("updated")
```

---

### Supporting Functions Used by `diff()`

#### `get_status()`

- Returns a tuple of lists: `(changed_paths, new_paths, deleted_paths)`.
- Determines changed files by comparing SHA-1 blob hashes of working copy files to those stored in the index.
- Filters working directory files, excluding `.git` directory contents.

#### `read_index()`

- Reads and parses the Git index file (`.git/index`).
- Returns a list of `IndexEntry` objects representing files staged in the index.

#### `read_object(sha1_prefix)`

- Given a SHA-1 prefix, locates the corresponding object file in the Git object store.
- Decompresses and parses the object, returning a tuple `(object_type, data_bytes)`.
- For blobs, `data_bytes` contains the file content.

#### `read_file(path)`

- Reads the raw bytes of a file from the working directory.

---

## ASCII Diagram: Diff Workflow

```
+-----------------+          +-----------------+          +-------------------+
|                 |  read    |                 |  read    |                   |
|     Git Index   +--------->+   Blob Object   +--------->+  Index File Data  |
|   (.git/index)  |          |  (compressed)   |          | (decoded as text) |
+-----------------+          +-----------------+          +-------------------+
         |                                                      ^
         |                                                      |
         |                                                      |
         |                                                      |
         |                                                      |
         v                                                      |
+-----------------+          +-----------------+               |
|                 |  read    |                 |               |
| Working Copy    +--------->+   Working File  |---------------+
|   (filesystem)  |          |  Content Bytes  |
+-----------------+          +-----------------+

                             |
                             | compare line-by-line using unified diff
                             v
                   +---------------------------+
                   |    Unified Diff Output    |
                   +---------------------------+
```

---

## Summary

The `diff()` function is a key utility that provides visibility into the local changes by showing differences between the staged snapshot (index) and the current working files. By combining index reading, object retrieval, and file reading with Python's diff utilities, it effectively presents a unified diff output familiar to Git users.

This functionality is tightly integrated with other core components such as the index management and object handling modules, fitting squarely into the "Working Copy Status and Diff" section of the project documentation.