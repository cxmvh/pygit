# pygit diff.md

## Overview

The `diff.md` file documents the mechanism by which the pygit library generates and displays diffs between files in the Git index and the working copy. It plays a critical role under the **Working Copy Status and Diff** section of the pygit project's documentation tree, complementing status reporting and file listing functionalities. This file explains how pygit identifies changed files by comparing the index and working directory versions, reads the corresponding blob objects, and produces unified diffs for human-readable output. The diff display helps users visually track changes before committing or pushing.

---

## Important Functions

### `diff()`

**Purpose:**  
Display diffs of files that have changed between the Git index and the working copy.

**Description:**  
The `diff()` function first retrieves the list of changed files by invoking `get_status()`. It then reads the current Git index entries to map file paths to their corresponding blob SHA-1 hashes. For each changed file, it:

1. Retrieves the blob object data from the index using the SHA-1 hash.
2. Decodes the blob data (the stored file content in the index) into lines.
3. Reads the working copy file content and splits it into lines.
4. Uses Python’s `difflib.unified_diff` to generate a unified diff between the indexed and working copy file contents.
5. Prints the diff to standard output.
6. Separates diffs of multiple files visually with a line of dashes.

**Usage Example:**

```python
# Show diffs for all files changed between index and working copy
diff()
```

**Sample Output:**

```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
 Line one
-Line two
+Line two modified
 Line three
+New line added
----------------------------------------------------------------------
```

---

### `get_status()`

**Purpose:**  
Determine the status of the working copy by comparing files in the working directory and the index.

**Description:**  
- Walks the working directory recursively (excluding `.git` directory) to collect all file paths.
- Reads the Git index entries to get tracked file paths and their blob hashes.
- Compares the blob hash of each file in the working directory with the corresponding index blob hash.
- Categorizes files into three lists:
  - `changed`: files tracked in the index but with differing content.
  - `new`: files present in the working directory but not in the index.
  - `deleted`: files tracked in the index but missing from the working directory.

**Returns:**  
A tuple `(changed_paths, new_paths, deleted_paths)` with sorted lists of file paths.

**Usage Example:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `read_index()`

**Purpose:**  
Read the Git index file and return a list of `IndexEntry` objects representing tracked files.

**Description:**  
- Opens and reads the `.git/index` file.
- Validates the index signature, version, and checksum.
- Parses individual entries containing file metadata, SHA-1 hashes, and file paths.
- Returns a list of entries sorted by paths.

**Usage Example:**

```python
entries = read_index()
for entry in entries:
    print(f"Tracked file: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### `read_object(sha1_prefix)`

**Purpose:**  
Read a Git object (blob, tree, commit) by its SHA-1 prefix and return its type and raw data.

**Description:**  
- Finds the object file path in `.git/objects` using the SHA-1 prefix.
- Decompresses the object file data.
- Parses the object header which specifies the type and size.
- Extracts the object data bytes.
- Verifies the size matches the header.
- Returns a tuple `(object_type, data_bytes)`.

**Usage Example:**

```python
obj_type, data = read_object('f572d396fae9206628714fb2ce00f72e94f2258f')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### `read_file(path)`

**Purpose:**  
Read the contents of a file from the working directory as bytes.

**Description:**  
- Opens the file in binary mode.
- Reads and returns the content bytes.

**Usage Example:**

```python
content_bytes = read_file('example.txt')
print(content_bytes.decode())
```

---

## How `diff()` Works: Detailed Flow and ASCII Diagram

```
+-----------------+      +-------------------+      +---------------------+
| get_status()    | ---> | read_index()      | ---> | For each changed     |
| (changed files) |      | (index entries)   |      | file path:           |
+-----------------+      +-------------------+      |                     |
                                              |     v                     v
                                              |  +-----------------+   +-----------------+
                                              |  | read_object()   |   | read_file()     |
                                              |  | (blob from index) |  | (working copy) |
                                              |  +-----------------+   +-----------------+
                                              |           |                   |
                                              |           v                   v
                                              |    index_lines           working_lines
                                              |      (list)               (list)
                                              |           \               /
                                              |            \             /
                                              |             v           v
                                              |          difflib.unified_diff()
                                              |                 |
                                              |                 v
                                              |           print diff lines
                                              |                 |
                                              +-----------------+
```

---

## Summary of the Diff Generation Process

1. **Identify Changed Files:**  
   `get_status()` compares working directory files with the index to find files that have been modified.

2. **Load Index Entries:**  
   `read_index()` loads metadata and blob hashes of tracked files.

3. **Retrieve Blob Data:**  
   For each changed file, the blob object stored in the Git object database is read via `read_object()`.

4. **Read Working Copy Content:**  
   The current version of the file in the working directory is read with `read_file()`.

5. **Generate Unified Diff:**  
   Use the Python `difflib.unified_diff()` to compute line-by-line differences between the indexed and working copy contents.

6. **Display Diff:**  
   Print the diff output showing deletions, additions, and context lines for each changed file.

---

## Additional Notes

- The diff output format follows the standard unified diff format familiar to Git users.
- The process assumes the index contains blob objects for tracked files.
- This diff functionality is foundational for staging and committing changes, enabling users to review modifications before committing or pushing.
- The diffs are printed to standard output but can be redirected or captured for other usage.

---

This concludes the `diff.md` documentation for pygit, explaining how diffs between the Git index and working copy files are generated and displayed.