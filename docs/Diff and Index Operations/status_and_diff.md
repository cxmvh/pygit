# status_and_diff.md

# Status and Diff Commands Documentation

---

## Overview

This document details the **status** and **diff** commands within the `pygit` project, focusing on how changes in the working directory and the Git index are detected and displayed. These commands are essential for users to understand the current state of their repository by highlighting modified, new, or deleted files, and by showing precise line-by-line differences between the working copy and the staged index. Positioned under the **Diff and Index Operations** section of the broader documentation tree, this file complements other index management and core Git operation documents by explaining how file changes are tracked and visualized.

---

## Command Functions and Usage

### 1. `status()`

#### Purpose

The `status()` function provides a summary of the working directory's current state relative to the Git index. It reports files that have been changed, newly added, or deleted. This command helps users quickly identify the files requiring attention before committing.

#### Parameters

- None

#### Operation Details

1. Calls `get_status()` to retrieve three lists:
   - `changed`: files modified since the last index update.
   - `new`: files present in the working directory but not in the index.
   - `deleted`: files removed from the working directory but still in the index.

2. Prints categorized lists:
   - "changed files:"
   - "new files:"
   - "deleted files:"

3. Each category is printed only if it contains files.

#### Example Usage

```python
>>> status()
changed files:
    README.md
    src/main.py
new files:
    docs/usage.md
deleted files:
    old_script.py
```

---

### 2. `diff()`

#### Purpose

The `diff()` function shows detailed line-by-line differences between the files staged in the Git index and their current state in the working directory. This command is crucial for reviewing exact changes before committing.

#### Parameters

- None

#### Operation Details

1. Calls `get_status()` to obtain the list of changed files.

2. Reads the current index entries via `read_index()` and maps them by file path.

3. For each changed file:
   - Reads the blob object from the Git object database corresponding to the staged version.
   - Reads the current working copy of the file.
   - Uses Python's `difflib.unified_diff()` to generate a unified diff output comparing the index version with the working copy.
   - Prints the diff lines to stdout.
   - Prints a separator line (`----------------------------------------------------------------------`) between diffs of multiple files, if applicable.

#### Example Usage

```python
>>> diff()
--- README.md (index)
+++ README.md (working copy)
@@ -1,3 +1,4 @@
-# pygit
+## pygit Project
+
+An implementation of Git in Python.
----------------------------------------------------------------------  
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -10,7 +10,9 @@
-    print("Hello, World!")
+    print("Hello, pygit!")
+    print("Status and diff commands updated.")
```

#### ASCII Diagram: Diff Flow

```
+----------------+          +------------------+          +-----------------------+
| Git Index File |  ---->   | Read Blob Object  |  ---->   | Staged file content    |
+----------------+          +------------------+          +-----------------------+
                                                                   |
                                                                   | Compare with
                                                                   v
                                                         +-----------------------+
                                                         | Working Directory File |
                                                         +-----------------------+
                                                                   |
                                                                   | Generate unified diff
                                                                   v
                                                         +-----------------------+
                                                         |     Diff Output        |
                                                         +-----------------------+
```

---

### 3. `get_status()`

#### Purpose

`get_status()` is a helper function that determines the status of files in the working directory by comparing them against the Git index. It categorizes files into changed, new, or deleted.

#### Parameters

- None

#### Operation Details

1. Recursively walks the working directory (excluding `.git`), collecting all file paths.

2. Reads the Git index using `read_index()` to obtain staged entries.

3. Compares working directory files and index entries:
   - `changed`: files in both sets but differing in content (based on blob SHA-1 hash).
   - `new`: files in working directory but not in index.
   - `deleted`: files in index but missing from working directory.

4. Returns the sorted lists `(changed, new, deleted)`.

#### Example Usage

```python
>>> changed, new, deleted = get_status()
>>> print("Changed:", changed)
Changed: ['README.md', 'src/main.py']
>>> print("New:", new)
New: ['docs/usage.md']
>>> print("Deleted:", deleted)
Deleted: ['old_script.py']
```

---

### 4. `read_index()`

#### Purpose

Reads the Git index file `.git/index` and returns a list of `IndexEntry` objects representing the current staged files and their metadata.

#### Parameters

- None

#### Operation Details

1. Reads the raw index file data.

2. Verifies the index file signature and version.

3. Parses entries sequentially, unpacking metadata and file paths.

4. Returns a list of `IndexEntry` objects.

#### Note

- If the index file does not exist, returns an empty list.

#### Example Usage

```python
>>> entries = read_index()
>>> for entry in entries:
...     print(entry.path, entry.sha1.hex())
README.md 9a3c2e4f...
src/main.py b1d48f9a...
```

---

### 5. `read_file(path)`

#### Purpose

Reads the content of a file from the working directory as bytes.

#### Parameters

- `path` (str): Path to the file.

#### Operation Details

- Opens the file in binary mode and reads its contents.

#### Example Usage

```python
>>> data = read_file('README.md')
>>> print(data.decode()[:60])
# pygit
This is a minimal Git implementation in Python.
```

---

### 6. `read_object(sha1_prefix)`

#### Purpose

Reads a Git object from the object database given a SHA-1 prefix, returning its type and raw data.

#### Parameters

- `sha1_prefix` (str): The prefix of the SHA-1 hash identifying the object.

#### Operation Details

1. Locates the object file using `find_object()`.

2. Reads and decompresses the object file.

3. Parses the header to extract the object type and size.

4. Returns a tuple `(object_type, data_bytes)`.

#### Example Usage

```python
>>> obj_type, data = read_object('9a3c2e4f')
>>> print(obj_type)
blob
>>> print(data.decode()[:50])
# pygit
This is a minimal Git implementation in Python.
```

---

## Related Concepts and Workflow

The `status` and `diff` commands operate closely with the Git index and the object database. The following ASCII diagram illustrates their interaction flow:

```
+--------------------+               +--------------------+
| Working Directory   |               |      Git Index     |
| (Current files)     |               | (Staged snapshots) |
+--------------------+               +--------------------+
          |                                  |
          |                                  |
          +------------+          +----------+
                       |          |
              get_status() compares files
                       |
      +----------------+----------------+
      |                                 |
 changed files                    new files / deleted files
      |                                 |
      v                                 v
  diff() shows line-by-line       status() prints summary
  differences for changed files
```

---

## Summary

- `status()` gives a high-level overview of file changes in the working directory.
- `diff()` provides detailed line-by-line differences for changed files.
- Both commands rely on `get_status()` to detect file changes by comparing the working directory and index.
- The index is read and parsed via `read_index()`.
- File contents and Git objects are accessed using `read_file()` and `read_object()` respectively.

Together, these commands enable users to track and visualize repository changes effectively before committing.

---

# End of status_and_diff.md documentation file content.