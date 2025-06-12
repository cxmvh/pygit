# status.md - Working Copy Status Functions

## Overview

The `status.md` documentation details the functions responsible for determining and displaying the current status of the working copy in a Git repository. This includes identifying files that have been changed, newly added, or deleted relative to the Git index. These functions serve as a crucial part of the "Working Copy Status and Diff" section in the overall documentation tree, enabling users and tools to inspect the state of the working directory and stage changes effectively.

This file ties together utilities for reading the index, hashing file contents, and comparing the working copy against the repository's index. Through these functions, users can gain clear insights into what modifications have taken place before committing or pushing changes.

---

## Function Documentation

### 1. `status()`

**Purpose:**  
Display the status of the working copy by listing changed, new, and deleted files compared to the Git index.

**Parameters:**  
None

**Operation Details:**  
- Calls `get_status()` to retrieve three lists: changed, new, and deleted file paths.
- Prints each category's files in a user-friendly format, only if files exist in that category.

**Example Usage:**
```python
status()
```
**Expected Output:**
```
changed files:
    src/main.py
new files:
    docs/README.md
deleted files:
    tests/old_test.py
```

---

### 2. `get_status()`

**Purpose:**  
Compute and return the current status of the working copy relative to the Git index, categorizing files as changed, new, or deleted.

**Returns:**  
A tuple of three lists:
- `changed_paths` (list of str): Files that exist both in the working directory and index but whose contents differ.
- `new_paths` (list of str): Files present in the working directory but not tracked in the index.
- `deleted_paths` (list of str): Files tracked in the index but missing in the working directory.

**Operation Details:**  
1. Recursively walks through the working directory (excluding `.git`), collecting all file paths.
2. Reads the Git index entries and maps them by path.
3. Compares file contents by hashing blobs for files present in both working directory and index.
4. Determines:
   - **Changed:** Files in both sets but differing contents.
   - **New:** Files only in working directory.
   - **Deleted:** Files only in the index.

**Example Usage:**
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### 3. `read_file(path)`

**Purpose:**  
Read the contents of a file at a given path as bytes.

**Parameters:**  
- `path` (str): Path to the file to read.

**Returns:**  
Bytes content of the file.

**Operation Details:**  
- Opens the file in binary mode and reads all bytes.

**Example Usage:**
```python
data = read_file('src/main.py')
print(data[:100])  # print first 100 bytes
```

---

### 4. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Compute the SHA-1 hash of object data of a given type, optionally writing the object to the Git object store.

**Parameters:**  
- `data` (bytes): Raw data to hash.
- `obj_type` (str): Type of Git object (`'blob'`, `'tree'`, `'commit'`, etc.).
- `write` (bool): Whether to write the object to the `.git/objects` directory.

**Returns:**  
Hex string of the SHA-1 hash.

**Operation Details:**  
- Constructs a Git object header with `<type> <size>\0`.
- Concatenates header and data, computes SHA-1 hash.
- If `write` is `True`, compresses and writes the object to `.git/objects/`.

**Example Usage:**
```python
with open('src/main.py', 'rb') as f:
    data = f.read()
sha1 = hash_object(data, 'blob')
print("Object hash:", sha1)
```

---

### 5. `read_index()`

**Purpose:**  
Read the Git index file and return a list of `IndexEntry` objects representing the tracked files.

**Returns:**  
List of `IndexEntry` instances.

**Operation Details:**  
- Reads `.git/index` file.
- Verifies checksum and header.
- Parses each index entry with metadata and path.
- Returns all entries.

**Example Usage:**
```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} - {entry.sha1.hex()}")
```

---

### 6. `add(paths)`

**Purpose:**  
Add specified file paths to the Git index, staging them for commit.

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**Operation Details:**  
- Normalizes path separators.
- Reads current index entries, excluding the ones to be replaced.
- For each path:
  - Reads file content and hashes it as a blob.
  - Retrieves file metadata.
  - Creates a new `IndexEntry`.
- Sorts entries and writes updated index back.

**Example Usage:**
```python
add(['src/main.py', 'docs/README.md'])
```

---

### 7. `diff()`

**Purpose:**  
Show unified diff output of files changed between the index and the working copy.

**Operation Details:**  
- Retrieves changed files via `get_status()`.
- For each changed file:
  - Reads blob data from index.
  - Reads current file content from working directory.
  - Uses `difflib.unified_diff` to generate the diff.
- Prints diffs separated by a line of dashes.

**Example Usage:**
```python
diff()
```

**Sample Output:**
```
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -1,4 +1,5 @@
-print("Hello, world!")
+print("Hello, Git world!")
+print("New line added")
----------------------------------------------------------------------
```

---

### 8. `ls_files(details=False)`

**Purpose:**  
List files currently in the Git index.

**Parameters:**  
- `details` (bool): If `True`, prints mode, SHA-1, and stage number alongside file paths.

**Operation Details:**  
- Reads index entries.
- Prints each entry's path or detailed info.

**Example Usage:**
```python
ls_files()
ls_files(details=True)
```

---

## ASCII Diagram: High-Level Working Copy Status Flow

```
+----------------------+
|   Working Directory   |  -- files present on disk, excludes .git
+----------+-----------+
           |
           v
+----------------------+
|     read_index()      |  -- reads tracked files from .git/index
+----------+-----------+
           |
           v
+----------------------------------------------+
| get_status(): Compare working dir and index  |
|  - Identify changed files                     |
|  - Identify new files                         |
|  - Identify deleted files                     |
+----------+-----------------------------------+
           |
           v
+----------------------+
|      status()        |  -- prints categorized file lists
+----------------------+
```

---

# Summary

The functions in `status.md` provide essential capabilities for inspecting the Git working copy state. These utilities rely on reading the index and working directory files, hashing blobs for content comparison, and presenting the differences clearly to the user. Together, they enable workflows that detect and stage changes efficiently, laying the foundation for committing and pushing updates in the repository.