# Working Copy Status (`status.md`)

## Overview

This document provides detailed reference information for obtaining and displaying the status of the working copy in a Git repository. It explains how to detect changed, new, and deleted files by comparing the working directory with the Git index. This file is part of the **Working Copy Status and Diff** section of the documentation tree and complements the `diff.md` file, which covers generating diffs between the index and working copy.

The status functionality is a crucial part of Git workflows because it informs users about the current state of their working copy relative to the repository's index. It identifies files that have been modified, added, or removed, enabling informed decisions for staging or committing changes.

---

## Function Documentation

### `status()`

#### Purpose

Displays the current status of the working copy by listing changed, new, and deleted files compared to the Git index.

#### Parameters

- None

#### Description

- Calls `get_status()` to get three lists: changed files, new files, and deleted files.
- Prints these lists to standard output, grouped by category, with appropriate headers.

#### Usage Example

```python
status()
```

Expected output might look like:

```
changed files:
    src/main.py
new files:
    docs/README.md
deleted files:
    old_script.py
```

---

### `get_status()`

#### Purpose

Determines the status of files in the working directory by comparing them to the Git index.

#### Returns

A tuple of three sorted lists:

- `changed_paths`: Files present both in the working directory and index but with differing contents.
- `new_paths`: Files present in the working directory but not in the index.
- `deleted_paths`: Files present in the index but missing in the working directory.

#### Description

1. Walks the current directory tree, excluding the `.git` directory, collecting paths of all files.
2. Reads the Git index entries and maps them by path.
3. Identifies:
   - Changed files by comparing blob hashes of working files and index entries.
   - New files as those found in the working directory but not in the index.
   - Deleted files as those in the index but not in the working directory.
4. Returns the three sets as sorted lists.

#### Example

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `read_file(path)`

#### Purpose

Reads the contents of a file at the specified path as bytes.

#### Parameters

- `path` (str): The file path to read.

#### Returns

- File contents as bytes.

#### Example

```python
data = read_file('src/main.py')
print(data[:100])  # Print first 100 bytes of file
```

---

### `hash_object(data, obj_type, write=True)`

#### Purpose

Computes the SHA-1 hash of given object data of a specified Git object type and optionally writes it to the object store.

#### Parameters

- `data` (bytes): Raw data of the object.
- `obj_type` (str): Type of the object (`blob`, `tree`, `commit`, etc.).
- `write` (bool): Whether to write the object to the `.git/objects` directory.

#### Returns

- SHA-1 hash of the object as a hex string.

#### Operation

- Constructs the Git object format: header (`<type> <size>\0`) + data.
- Computes SHA-1 digest of this format.
- If `write` is `True`, compresses and writes the object to `.git/objects/<sha1-prefix>/<sha1-suffix>`.
- Returns the SHA-1 hex string.

#### Example

```python
data = read_file('src/main.py')
sha1 = hash_object(data, 'blob')
print(f"Object stored with SHA-1: {sha1}")
```

---

### `read_index()`

#### Purpose

Reads the `.git/index` file and returns a list of `IndexEntry` objects representing the index state.

#### Returns

- List of `IndexEntry` objects with file metadata and SHA-1 hashes.

#### Description

- Reads the binary index file and validates its checksum.
- Parses header and entries according to Git index format version 2.
- Extracts file metadata and paths.
- Returns the list of entries.

#### Example

```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} ({entry.sha1.hex()})")
```

---

### `add(paths)`

#### Purpose

Adds specified file paths to the Git index by hashing their contents and updating index entries.

#### Parameters

- `paths` (list of str): File paths to add.

#### Description

- Normalizes path separators to Unix style.
- Reads existing index entries excluding the given paths.
- For each new path:
  - Reads file content.
  - Hashes content as a blob.
  - Creates a new `IndexEntry` with file metadata.
- Combines and sorts all entries by path.
- Writes updated list to the index file.

#### Example

```python
add(['src/main.py', 'docs/README.md'])
```

---

### `diff()`

#### Purpose

Shows the unified diff between the index and the working copy for changed files.

#### Description

- Uses `get_status()` to find changed files.
- For each changed file:
  - Reads blob data from the index.
  - Reads current file content from working directory.
  - Generates and prints a unified diff.

#### Example

```python
diff()
```

Output resembles standard git diff output, e.g.:

```
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -1,4 +1,6 @@
 def main():
-    print("Hello World")
+    print("Hello, Git!")
+    print("New line added")
```

---

## ASCII Diagram: Status Detection Workflow

```
+-------------------+
| Working Directory  |
| (files on disk)    |
+---------+---------+
          |
          | Read all files (excluding .git)
          v
+-------------------+
| Set of file paths  |
+---------+---------+
          |
          | Compare with
          v
+-------------------+
| Git Index entries  |
+---------+---------+
   |       |       |
   |       |       |
 Changed  New    Deleted
 Files   Files   Files
   |       |       |
   +-------+-------+
           |
           v
    Display Status
```

---

## Related Documentation

- [Index Management / Index File Format (`index.md`)](../Index%20Management/index.md)
- [Working Copy Status and Diff / Diff Generation (`diff.md`)](diff.md)

---

# Summary

This documentation file covers the essential functions and procedures used to determine and display the status of files in the working copy relative to the Git index. It explains how files are classified as changed, new, or deleted, and provides usage examples for each function involved in this process. The status functionality serves as a foundation for higher-level Git operations such as staging changes, committing, and generating diffs.