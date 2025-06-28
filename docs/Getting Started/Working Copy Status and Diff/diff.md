# diff.md

# Showing diffs of changed files between index and working copy

---

## Overview

The `diff.md` file documents the functionality related to displaying differences ("diffs") between the Git index and the working copy files in the `pygit` repository. This feature is crucial for developers to inspect changes that have been made locally but are not yet staged or committed. It fits within the broader **Working Copy Status and Diff** section of the documentation tree, which provides tools for inspecting the working directory and comparing it against the Git index. The `diff` functionality complements status reporting (`status.md`) and is used during workflows involving staging (`add.md`) and committing (`commit.md`). This document provides detailed insights into the important functions involved in generating and showing diffs, with usage examples and explanations.

---

## Important Functions

### 1. `diff()`

#### Purpose

Show the line-by-line unified diff of files that have changed between the Git index and the working copy. This function highlights modifications that exist in the working directory but are not yet staged or committed.

#### How it Works

- Retrieves the list of changed files by comparing the working directory and the index (`get_status()`).
- Reads the current index entries (`read_index()`) and creates a mapping of paths to index entries.
- For each changed file:
  - Retrieves the blob object SHA-1 from the index.
  - Reads the blob object data for the file contents stored in the index.
  - Reads the file contents from the working copy.
  - Uses Python's `difflib.unified_diff` to generate a unified diff between the indexed file content and the working copy content.
  - Prints the diff output to standard output.
  - Prints a separator line (`'--------------------------------------------------------------------'`) between diffs of multiple files.

#### Usage Example

```python
from pygit import diff

# Show diffs of all changed files between the index and working copy
diff()
```

#### Output Example

```
--- example.txt (index)
+++ example.txt (working copy)
@@ -1,3 +1,4 @@
 Line 1
-Line 2
+Modified Line 2
 Line 3
+Added Line 4
--------------------------------------------------------------------
```

---

### 2. `get_status()`

#### Purpose

Determine the status of the working copy by identifying files that have changed, are new, or have been deleted when compared to the Git index.

#### How it Works

- Walks through the working directory recursively, ignoring the `.git` directory, and collects all file paths.
- Reads the Git index entries and maps them by file path.
- Compares file contents in the working directory with the blobs stored in the index by hashing the working copy files.
- Categorizes files into:
  - **Changed:** Files existing in both working directory and index but with different content.
  - **New:** Files present in working directory but not in index.
  - **Deleted:** Files present in index but missing in working directory.

#### Returns

A tuple of three sorted lists:

```python
(changed_paths, new_paths, deleted_paths)
```

#### Usage Example

```python
from pygit import get_status

changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 3. `read_index()`

#### Purpose

Read the Git index file (`.git/index`) and parse it into a list of `IndexEntry` objects representing the current staging area.

#### How it Works

- Opens and reads the `.git/index` file.
- Validates the file signature (`DIRC`) and version (expected `2`).
- Verifies the SHA-1 checksum of the index file.
- Parses each index entry including metadata (ctime, mtime, device, inode, mode, uid, gid, size, SHA-1, flags) and the file path.
- Returns all parsed entries as a list.

#### Usage Example

```python
from pygit import read_index

entries = read_index()
for entry in entries:
    print(f"{entry.path} - SHA1: {entry.sha1.hex()}")
```

---

### 4. `read_object(sha1_prefix)`

#### Purpose

Read a Git object by its SHA-1 hash prefix, returning the object type and raw data.

#### How it Works

- Locates the object file in `.git/objects` using the SHA-1 prefix.
- Decompresses the object data using zlib.
- Parses the header to extract object type and size.
- Returns a tuple `(obj_type, data)` where:
  - `obj_type` is a string like `'blob'`, `'commit'`, or `'tree'`.
  - `data` is the raw byte content of the object.

#### Usage Example

```python
from pygit import read_object

obj_type, data = read_object('a1b2c3d4')
print(f"Object type: {obj_type}")
print(data.decode())
```

---

### 5. `read_file(path)`

#### Purpose

Read the contents of a file from the working directory as bytes.

#### Usage Example

```python
from pygit import read_file

content = read_file('example.txt')
print(content.decode())
```

---

### 6. `hash_object(data, obj_type, write=True)`

#### Purpose

Compute the SHA-1 hash of an object given its data and type, optionally writing the object to the Git object store.

#### How it Works

- Constructs the object header: `"<obj_type> <size>\0"`.
- Concatenates header and data.
- Computes SHA-1 hash of this full data.
- If `write` is `True`, compresses and writes the object to `.git/objects/<sha1_prefix>/<sha1_suffix>`.
- Returns the SHA-1 hash as a hexadecimal string.

#### Usage Example

```python
from pygit import hash_object

data = b"Hello, Git!"
sha1 = hash_object(data, "blob")
print(f"Object stored with SHA-1: {sha1}")
```

---

### 7. `add(paths)`

#### Purpose

Add files specified by `paths` to the Git index by hashing their contents and updating the index entries.

#### How it Works

- Reads the current index entries.
- Removes entries with paths that are being added (to avoid duplicates).
- For each path:
  - Reads and hashes the file contents as a blob.
  - Collects file metadata (ctime, mtime, mode, size, etc.).
  - Creates a new `IndexEntry`.
- Sorts all entries by path.
- Writes the updated list back to the index.

#### Usage Example

```python
from pygit import add

add(['example.txt', 'README.md'])
```

---

## ASCII Diagram: Diff Workflow Overview

```
Working Copy                         Git Index
  (Files on disk)                      (Staged files)

     +----------------+                +----------------+
     |                |                |                |
     |  example.txt   |                |  example.txt   |
     |  (modified)    |                |  (blob object) |
     |                |                |                |
     +----------------+                +----------------+
             |                               |
             |                               |
             +----------- diff() ------------+
                         compares

  diff() generates unified diff output showing line-by-line changes
  between the 'example.txt' in working copy and the blob stored in index.
```

---

## Summary

The `diff.md` documentation explains how diffs are generated and displayed by comparing the current working copy files against the staged snapshots in the Git index. Using core functions such as `diff()`, `get_status()`, `read_index()`, and `read_object()`, `pygit` enables users to inspect exactly what changes have been made locally before staging or committing them. This functionality is essential for version control workflows and integrates tightly with other commands such as `add`, `status`, and `commit`.