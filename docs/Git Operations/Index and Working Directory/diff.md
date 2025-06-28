# diff.md - Showing Diffs Between Index and Working Copy Files

## Overview

The `diff.md` file documents how to display the differences between files in the Git index and the working copy in the `pygit` repository. This functionality is crucial for understanding what changes have been made locally but not yet staged or committed. It fits into the broader documentation section "Working Copy Status and Diff," which provides tools for inspecting the working copy and differences with the index. The diff feature complements status reporting and file listing, enhancing a developer's ability to track and review changes before committing or pushing them.

This document details the primary function `diff()` responsible for showing diffs, along with essential supporting functions such as `get_status()`, which detects changed files, and `read_object()`, which reads Git objects by their SHA-1 hashes. Usage examples and explanations are provided for each function to guide users on how to utilize these features effectively.

---

## Function: diff()

### Purpose

`diff()` displays the unified diff output for all files that have changed between the Git index and the working copy. It compares the blob data stored in the index with the current contents of the working copy files and prints a line-by-line diff to the standard output.

This function is critical for reviewing local modifications before staging or committing changes.

### Parameters

- This function takes no parameters.

### Description

- Calls `get_status()` to obtain the list of changed files.
- Reads the index entries into a dictionary for quick lookup by file path.
- For each changed file:
  - Reads the blob object data from the index (the "old" version).
  - Reads the current working copy file content (the "new" version).
  - Uses Python's `difflib.unified_diff` to generate a unified diff between these two versions.
  - Prints the diff output.
- Prints a separator line (`----------------------------------------------------------------------`) between diffs of multiple files for readability.

### Example Usage

```python
import pygit

# Show diffs of all changed files between index and working copy
pygit.diff()
```

### Output Example

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

--- src/main.py (index)
+++ src/main.py (working copy)
@@ -10,7 +10,7 @@
-def old_function():
+def new_function():
     pass
```

---

## Function: get_status()

### Purpose

`get_status()` determines which files in the working directory have changed relative to the Git index. It returns three lists identifying:

- Files that have changed (modified).
- New files (untracked files not in the index).
- Deleted files (tracked files removed from the working directory).

### Parameters

- This function takes no parameters.

### Description

- Recursively walks the working directory tree, excluding the `.git` directory.
- Collects all file paths currently present.
- Reads the Git index entries and maps them by file path.
- Compares the working copy files against the index entries to detect:
  - Changed files where the blob hash differs.
  - New files that exist in the working directory but not in the index.
  - Deleted files that exist in the index but not in the working directory.
- Returns sorted lists of changed, new, and deleted file paths.

### Example Usage

```python
changed, new, deleted = pygit.get_status()

print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

## Function: read_object(sha1_prefix)

### Purpose

`read_object()` reads a Git object by its SHA-1 prefix and returns its type and data payload.

This allows access to the contents of blobs, commits, trees, etc., stored in the `.git/objects` directory.

### Parameters

- `sha1_prefix` (str): The SHA-1 prefix (at least 2 characters) identifying the object.

### Description

- Uses `find_object()` to locate the file path of the object in the object store.
- Reads and decompresses the object data.
- Parses the header to extract the object type and size.
- Returns a tuple `(object_type, data_bytes)`.

### Example Usage

```python
obj_type, data = pygit.read_object('a1b2c3d4')

print("Object type:", obj_type)
print("Object data (first 100 bytes):", data[:100])
```

---

## Function: read_index()

### Purpose

`read_index()` reads the Git index file and returns a list of `IndexEntry` objects representing the entries currently staged.

### Parameters

- None.

### Description

- Reads the `.git/index` file.
- Verifies the checksum to ensure file integrity.
- Parses the header and index entries.
- Returns a list of `IndexEntry` objects, each containing metadata and SHA-1 of the indexed file.

### Example Usage

```python
entries = pygit.read_index()

for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

## Function: find_object(sha1_prefix)

### Purpose

`find_object()` locates the path of a Git object file on disk given its SHA-1 prefix.

### Parameters

- `sha1_prefix` (str): The prefix of the SHA-1 hash identifying the object.

### Description

- Validates the prefix length (at least 2 characters).
- Navigates the `.git/objects` directory structure.
- Finds matching object files starting with the given prefix.
- Raises an error if no or multiple objects match.
- Returns the full file path to the object.

### Example Usage

```python
path = pygit.find_object('a1b2')

print("Object file path:", path)
```

---

## ASCII Diagram: Relationship of Working Copy, Index, and Objects in Diff

```
Working Copy Files
      |
      | (read current file contents)
      v
+--------------------+     +----------------------+
|                    |     |                      |
|    Git Index       | --> |    Git Object Store   |
| (IndexEntry blobs) |     | (Compressed objects)  |
|                    |     |                      |
+--------------------+     +----------------------+
      ^
      | (diff compares blob data vs. file contents)
      |
   diff() function
```

This diagram illustrates that the `diff()` functionality compares the content of files in the working copy with the blobs referenced in the Git index, which in turn is backed by the Git object store.

---

# Summary

The `diff.md` file documents the mechanism by which `pygit` shows differences between files staged in the index and the local working copy. The core function `diff()` leverages `get_status()` to discover changed files and reads Git blob objects to generate unified diffs against the working copy contents. This feature is essential for developers to review local changes before staging or committing them, providing fine-grained insight into modifications at the file content level.