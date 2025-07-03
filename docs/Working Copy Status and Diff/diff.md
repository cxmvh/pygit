# diff.md

# Generating Diffs Between Index and Working Copy

## Overview

This document describes how diffs are generated between the Git index and the working copy in the repository. It belongs to the **Working Copy Status and Diff** section of the documentation tree, focusing on commands and functions that detect changes in files, compare contents, and present differences to the user.

The primary functionality covered here is the `diff()` function, which identifies files that have changed between the index (staged snapshot) and the current working directory, then produces unified diffs displaying line-by-line differences. This feature is essential for reviewing modifications before committing changes.

---

## Function: diff()

### Purpose

`diff()` displays the differences between the files recorded in the Git index and the files in the working copy. It shows line-by-line changes for all files that have been modified since the last index update.

### Parameters

- This function takes no parameters.

### Behavior and Steps

1. Calls `get_status()` to obtain lists of changed, new, and deleted files in the working directory compared to the index.
2. Reads the Git index entries via `read_index()` and builds a dictionary mapping file paths to their index entries.
3. Iterates over all changed files:
   - Retrieves the SHA-1 hash of the blob object stored in the index for the file.
   - Reads the blob object from the Git object database using `read_object()` and decodes its content into lines.
   - Reads the corresponding file from the working directory and splits its content into lines.
   - Uses Python's `difflib.unified_diff` to generate a unified diff between the index version and the working copy version.
   - Prints the diff output with headers indicating the file and version.
4. Prints a separator line of dashes (`-`) between diffs for multiple changed files.

### Example Usage

```python
diff()
```

This will output diffs for all changed files, for example:

```
--- README.md (index)
+++ README.md (working copy)
@@ -1,3 +1,4 @@
 Hello World!
-This is a sample README.
+This is an updated sample README.
+Additional line added.
----------------------------------------------------------------------
```

---

## Supporting Functions

### read_file(path)

Reads the contents of a file at the specified path as bytes.

- **Parameters:**
  - `path` (str): The file path to read.
- **Returns:** Bytes content of the file.

**Example:**

```python
data_bytes = read_file('README.md')
print(data_bytes.decode())
```

---

### read_object(sha1_prefix)

Reads a Git object identified by a SHA-1 prefix and returns a tuple of `(object_type, data_bytes)`.

- **Parameters:**
  - `sha1_prefix` (str): The SHA-1 prefix of the object to read.
- **Returns:** Tuple `(obj_type, data)` where `obj_type` is the object type string (e.g., `'blob'`) and `data` is the raw byte content.
- **Raises:** `ValueError` if the object is not found or the prefix is ambiguous.

**Steps:**

1. Locates the full object file path using `find_object(sha1_prefix)`.
2. Reads and decompresses the object file using zlib.
3. Parses the header for the object type and size.
4. Verifies that the declared size matches the actual data length.
5. Returns the object type and data.

**Example:**

```python
obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### read_index()

Reads the current Git index file and returns a list of `IndexEntry` objects representing tracked files.

- **Returns:** List of `IndexEntry` instances.
- **Notes:** Returns an empty list if the index file does not exist.

**Process:**

- Reads the `.git/index` file.
- Validates its checksum.
- Parses each index entry including metadata and file path.
- Returns all entries as objects.

**Example:**

```python
entries = read_index()
for entry in entries:
    print(f"{entry.path}: {entry.sha1.hex()}")
```

---

### get_status()

Determines the status of the working directory relative to the index.

- **Returns:** Tuple `(changed_paths, new_paths, deleted_paths)`:
  - `changed_paths`: List of file paths modified but tracked.
  - `new_paths`: List of new untracked file paths.
  - `deleted_paths`: List of paths deleted from the working directory but tracked in index.

**How it works:**

- Walks the working directory excluding `.git`.
- Compares files with index entries by hashing contents.
- Categorizes files accordingly.

**Example:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

## ASCII Diagram: Diff Generation Flow

```
+-------------------+
|    diff() called  |
+---------+---------+
          |
          v
+----------------------------+
|   get_status() called      |
| - Lists changed files      |
+------------+---------------+
             |
             v
+----------------------------+
|   read_index() to get      |
|   index entries dict       |
+------------+---------------+
             |
             v
+----------------------------+
| For each changed file:     |
| - read_object() blob from  |
|   index                    |
| - read_file() from working |
|   copy                     |
| - difflib.unified_diff()   |
| - print diff lines         |
+----------------------------+
```

---

## Summary

The `diff()` function provides developers a clear, line-by-line comparison of changes made in the working directory relative to the last staged snapshot in the index. It leverages several core utility functions for reading files, Git objects, and the index, integrating these to produce human-readable diffs similar to `git diff`.

For more details on related status commands, see [status.md](./status.md). For index management, see [index.md](./index.md). For Git object handling, refer to [objects.md](./objects.md).