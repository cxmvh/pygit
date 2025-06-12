# diff.md

# Viewing Diffs Between the Index and Working Copy Files

---

## Overview

This document explains how to view differences ("diffs") between the Git index (staging area) and the working copy files in a repository. It is part of the **Working Copy Status and Diff** section of the overall Git documentation tree, which deals with inspecting and managing the current state of files relative to the repository’s index and HEAD commit. Understanding how to see these diffs is crucial for developers to review changes before committing, ensuring only intended modifications are staged and committed.

The file `diff.md` primarily documents the `diff()` function and related utilities that enable comparing the content stored in the index with the current working files, producing output similar to `git diff` in official Git. This functionality supports visualizing line-by-line changes, helping developers track modifications effectively.

---

## Important Functions

### 1. `diff()`

#### Purpose

`diff()` shows the unified diff of files that have been changed between the Git index and the working copy. It compares the staged version (index) of each changed file with the current working copy, highlighting additions, deletions, and modifications.

#### Parameters

- None

#### Preconditions

- The repository must have an initialized `.git` directory with an index file.
- The working copy contains files possibly modified compared to the index.

#### Operation Details

1. Calls `get_status()` to obtain lists of changed, new, and deleted files; focus is on changed files.
2. Reads the current index entries using `read_index()` and creates a lookup dictionary mapping file paths to their index entries.
3. For each changed file path:
   - Retrieves the SHA-1 hash of the blob stored in the index.
   - Reads the object data (blob) from the object store using `read_object()`.
   - Decodes the blob data from the index and splits into lines (`index_lines`).
   - Reads the working copy file content and splits into lines (`working_lines`).
   - Uses Python’s `difflib.unified_diff()` to generate a unified diff between the index and working copy lines.
   - Prints the diff lines to standard output.
4. If multiple changed files exist, separates diffs with a dashed line for clarity.

#### Example Usage

```python
diff()
```

**Output Example:**

```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
 Line 1
 Line 2
+Added line 3
 Line 4
----------------------------------------------------------------------
```

#### ASCII Diagram: Comparison Flow

```
+-------------------+        +---------------------+        +----------------------+
|                   |        |                     |        |                      |
|  Git Index Blob   | -----> |  read_object(sha1)   |        |  Working Copy File    |
|  (staged version) |        |  decompress & parse  |        |  (current file on disk)|
|                   |        |                     |        |                      |
+-------------------+        +---------------------+        +----------------------+
           |                             |                            |
           |                             |                            |
           |                             +------------+               |
           |                                          |               |
           +------------------------------------------+---------------+
                                                      |
                                                      v
                                           difflib.unified_diff()
                                                      |
                                                      v
                                            Output diff lines on stdout
```

---

### 2. `read_file(path)`

#### Purpose

Reads and returns the contents of a file from the working copy as bytes.

#### Parameters

- `path` (str): Path to the file relative to the repository root.

#### Operation Details

- Opens the file in binary mode.
- Reads all bytes and returns them.

#### Example Usage

```python
data_bytes = read_file('src/main.py')
```

---

### 3. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the object store given its SHA-1 prefix and returns a tuple containing the object type and its raw data bytes.

#### Parameters

- `sha1_prefix` (str): The SHA-1 hash prefix (minimum 2 characters) identifying the object.

#### Operation Details

1. Finds the full object file path in `.git/objects/` using the SHA-1 prefix via `find_object()`.
2. Reads the compressed object file and decompresses it using `zlib`.
3. Parses the object header to extract the object type (e.g., `blob`) and size.
4. Extracts the object data bytes.
5. Verifies the size matches the expected size.
6. Returns `(object_type, data)`.

#### Example Usage

```python
obj_type, data = read_object('4a2c7e')
print(obj_type)  # 'blob'
print(data.decode())
```

---

### 4. `get_status()`

#### Purpose

Determines and returns the status of the working copy relative to the index: which files are changed, new, or deleted.

#### Returns

Tuple of three lists:

- `changed_paths`: Paths of files present in index and working copy but with differing content.
- `new_paths`: Paths of files present in working copy but not in index.
- `deleted_paths`: Paths of files present in index but missing from working copy.

#### Operation Details

1. Walks the working directory (excluding `.git`).
2. Collects all file paths in the working copy.
3. Reads index entries and maps them by path.
4. Computes:
   - Changed files by comparing blob hashes of working files and index entries.
   - New files as those in working copy but not in index.
   - Deleted files as those in index but not in working copy.

#### Example Usage

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### Related Functions Summary

- `read_index()`: Reads the Git index file and returns the list of index entries.
- `hash_object(data, obj_type, write=True)`: Computes SHA-1 hash of object data and optionally writes to object store.
- `find_object(sha1_prefix)`: Locates object file path from a SHA-1 prefix.
- `write_file(path, data)`: Writes data to a file, used internally when writing Git objects.

---

## Additional Notes

- The diff output closely follows the standard unified diff format, enabling easy reading and integration with tools.
- The `diff()` function relies on the index accurately reflecting the staged versions.
- This approach internally mimics `git diff` functionality but is simplified and may not handle all edge cases or binary files.

---

## Summary Diagram: Diff Process

```
Working Copy Files           Git Index (Staged Blobs)
        |                            |
        |                            |
        v                            v
  Read file content           Read index entries
        |                            |
        +---- Compare hashes --------+
        |                            |
 Changed files                     Blob objects
        |                            |
        +-------- read_object --------+
        |                            |
   Working file lines          Index blob lines
        |                            |
        +------- difflib.unified_diff ------+
                                       |
                               Print diff output
```

---

This documentation equips users and developers with a comprehensive understanding of how diffs are generated between the staged index and the working copy, providing both conceptual insight and practical usage examples for the core functions involved.