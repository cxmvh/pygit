# index.md - Git Index File Management

## Overview

This document provides a comprehensive reference for functions involved in managing the Git index file, a core component in Git's staging area. The index file serves as a cache between the working directory and the repository, tracking file states and enabling efficient commit operations. Functions described herein include reading and writing the index file, adding files to the index, and listing index entries.

Situated within the broader "Index and Working Copy Management" section of the documentation, these functions integrate with repository initialization and core Git commands to facilitate status checking, diffing, staging, and committing changes. They are pivotal for commands such as `git add`, `git status`, and `git commit`, as implemented in the `pygit` project.

---

## Function Documentation

---

### `read_index()`

**Purpose:**  
Reads the Git index file located at `.git/index` and returns a list of `IndexEntry` objects representing the staged files.

**Parameters:**  
_None_

**Returns:**  
- `List[IndexEntry]`: List of index entries parsed from the index file.

**Operation:**

1. Opens and reads the raw contents of `.git/index`.
2. Verifies the file's SHA-1 checksum to ensure integrity.
3. Parses the index header to extract signature, version, and number of entries.
4. Iteratively unpacks each entry's fixed-size fields and variable-length path.
5. Constructs `IndexEntry` objects and collects them into a list.
6. Returns the list of entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}, Mode: {oct(entry.mode)}")
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file, updating the staging area.

**Parameters:**  
- `entries (List[IndexEntry])`: The entries to write into the index.

**Returns:**  
_None_

**Operation:**

1. Packs each `IndexEntry` into the binary index file format, ensuring proper alignment.
2. Constructs the index header with signature, version, and entry count.
3. Concatenates entries and appends a SHA-1 checksum for integrity.
4. Writes the complete byte sequence to `.git/index`.

**Example Usage:**
```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### `add(paths)`

**Purpose:**  
Adds one or more file paths from the working directory to the Git index, staging them for commit.

**Parameters:**  
- `paths (List[str])`: List of file paths to add to the index.

**Returns:**  
_None_

**Operation:**

1. Normalize paths to use forward slashes.
2. Reads the current index entries.
3. Filters out any existing entries matching the paths to be added.
4. For each new path:
   - Reads the file content.
   - Computes the SHA-1 hash of the file blob.
   - Collects file metadata (timestamps, device, inode, mode, UID, GID, size).
   - Creates a new `IndexEntry` with this information.
5. Appends new entries, sorts all entries by path, and writes back to the index.

**Example Usage:**
```python
add(['src/main.py', 'README.md'])
```

---

### `ls_files(details=False)`

**Purpose:**  
Lists the files currently staged in the Git index.

**Parameters:**  
- `details (bool)`: If `True`, prints detailed information including mode, SHA-1, and stage number; otherwise, prints only the paths.

**Returns:**  
_None_

**Operation:**

1. Reads the index entries.
2. Iterates over each entry:
   - If `details` is `True`, prints mode (octal), SHA-1 hash, stage (from flags), and path.
   - Otherwise, prints only the file path.

**Example Usage:**
```python
ls_files(details=True)
```

**Output Example:**
```
100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0   README.md
100755 3f8a1f4d8a6825a6a0b2e2d1e3f6e1c9f1a07f8a 0   src/main.py
```

---

### `status()`

**Purpose:**  
Displays the status of the working copy relative to the Git index, listing changed, new, and deleted files.

**Parameters:**  
_None_

**Returns:**  
_None_

**Operation:**

1. Calls `get_status()` to determine changed, new, and deleted files.
2. Prints categorized lists of these files.

**Example Usage:**
```python
status()
```

**Output Example:**
```
changed files:
    src/main.py
new files:
    docs/usage.md
deleted files:
    old_script.py
```

---

### `get_status()`

**Purpose:**  
Determines the current status of files in the working directory with respect to the index.

**Parameters:**  
_None_

**Returns:**  
- Tuple of lists `(changed_paths, new_paths, deleted_paths)`.

**Operation:**

1. Recursively walks the working directory, ignoring `.git` directory.
2. Collects all file paths in working directory.
3. Reads index entries and maps paths to entries.
4. Computes:
   - `changed`: Files present in both but with different contents.
   - `new`: Files present in working directory but not in index.
   - `deleted`: Files present in index but missing in working directory.

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### `diff()`

**Purpose:**  
Shows the unified diff of files changed between the Git index and the working copy.

**Parameters:**  
_None_

**Returns:**  
_None_

**Operation:**

1. Retrieves changed files from `get_status()`.
2. For each changed file:
   - Reads the blob data from the index.
   - Reads the working copy file.
   - Uses Python's `difflib.unified_diff` to compute the diff.
3. Prints the diff output, separated by lines for multiple files.

**Example Usage:**
```python
diff()
```

**Sample Output:**
```
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -1,3 +1,4 @@
 print("Hello, world!")
+print("New line added")
----------------------------------------------------------------------  
```

---

### `read_file(path)`

**Purpose:**  
Reads the contents of a given file as bytes.

**Parameters:**  
- `path (str)`: File path.

**Returns:**  
- `bytes`: File contents.

**Example Usage:**
```python
data = read_file('README.md')
print(data.decode())
```

---

### `write_file(path, data)`

**Purpose:**  
Writes byte data to a specified file path.

**Parameters:**  
- `path (str)`: Target file path.
- `data (bytes)`: Data to write.

**Returns:**  
_None_

**Example Usage:**
```python
write_file('.git/index', index_data)
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of a Git object (blob, tree, commit) and optionally writes it to the object store.

**Parameters:**  
- `data (bytes)`: Raw object data.
- `obj_type (str)`: Type of object (`'blob'`, `'tree'`, `'commit'`).
- `write (bool)`: If `True`, write the compressed object to the `.git/objects` directory.

**Returns:**  
- `str`: Hexadecimal SHA-1 hash of the object.

**Example Usage:**
```python
sha1 = hash_object(b'hello world\n', 'blob')
print(sha1)
```

---

### ASCII Diagram: Git Index File Structure

```
+----------------+------------------+--------------------------+
| Header (12 bytes) | Entry 1 (variable) | Entry 2 (variable) | ... |
+----------------+------------------+--------------------------+
| Signature: 4 bytes 'DIRC'                         |
| Version: 4 bytes (e.g., 2)                        |
| Number of entries: 4 bytes                        |
+--------------------------------------------------+
| Entry:                                           |
|  +------------------+---------------------------+
|  | Fixed fields (62 bytes)                      |
|  | Path (variable length, null-terminated)     |
|  | Padding (to 8-byte alignment)                |
|  +----------------------------------------------+
+--------------------------------------------------+
| SHA-1 checksum (20 bytes) of all preceding data  |
+--------------------------------------------------+
```

---

## Summary

The functions documented here form the backbone of Git's staging area manipulation within the `pygit` project. They enable reading and writing the index file, managing staged files, assessing repository status, and preparing data for commit objects. Mastery of these functions is essential for understanding how Git tracks and stages changes efficiently.

For further details on related topics such as repository initialization and commit creation, see the linked documentation files in the "Index and Working Copy Management" and "Core Git Commands" sections.