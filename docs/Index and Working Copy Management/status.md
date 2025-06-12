# status.md - Checking Working Copy Status, File Changes, and Reporting

---

## Overview

The `status.md` documentation details the mechanisms and functions used to inspect the current working copy of a Git repository managed by the `pygit` implementation. This file explains how the system detects changes between the working directory and the Git index, identifies new and deleted files, and reports these statuses to the user. It is part of the **Index and Working Copy Management** section within the broader documentation tree, emphasizing its role in tracking file states and supporting commands like `git status` and `git diff`.

The functionalities covered here are crucial for users to understand the state of their working directory relative to the repository's index, enabling informed decisions before staging, committing, or pushing changes.

---

## Function Documentation

### 1. `status()`

**Purpose:**  
Displays a summary of the working copy’s current status by listing changed, new, and deleted files compared to the Git index.

**Parameters:**  
None

**Returns:**  
None (prints output directly)

**Operation:**  
- Calls `get_status()` to retrieve three lists: changed files, new files, and deleted files.
- Prints each category if non-empty, listing file paths indented for readability.

**Example Usage:**  
```python
status()
```
**Sample Output:**
```
changed files:
    README.md
    src/main.py
new files:
    docs/usage.md
deleted files:
    old_script.py
```

---

### 2. `get_status()`

**Purpose:**  
Computes the status of the working copy by comparing the files in the working directory against those recorded in the Git index.

**Parameters:**  
None

**Returns:**  
A tuple of three sorted lists:
- `changed_paths`: Files present in both working directory and index but with differing content.
- `new_paths`: Files present in working directory but not in index.
- `deleted_paths`: Files present in index but missing from working directory.

**Preconditions:**  
- Must be run inside a Git repository root.
- The Git index file exists or is empty.

**Operation Steps:**  
1. Recursively walks the working directory tree (excluding `.git` directory), collecting paths of all files.
2. Reads the index entries using `read_index()`, mapping paths to their entries.
3. Determines which files:
   - Exist in both places but differ in content by hashing working copy files and comparing to index SHA-1.
   - Exist only in working directory (new files).
   - Exist only in index (deleted files).
4. Returns the paths sorted alphabetically.

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
Reads the full content of a file from the filesystem as bytes.

**Parameters:**  
- `path` (str): Path to the file to read.

**Returns:**  
- Bytes content of the file.

**Operation:**  
- Opens the file in binary mode.
- Reads and returns all content.

**Example Usage:**  
```python
data = read_file('README.md')
print(data.decode())  # prints file content as string
```

---

### 4. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of the given data as a Git object of the specified type, optionally writing it to the Git object store.

**Parameters:**  
- `data` (bytes): Raw data to hash.
- `obj_type` (str): Git object type, e.g., `'blob'`, `'tree'`, `'commit'`.
- `write` (bool): If `True`, writes compressed object data to `.git/objects`.

**Returns:**  
- SHA-1 hash string of the object.

**Operation Steps:**  
1. Constructs a header: `<obj_type> <length>` followed by a null byte.
2. Concatenates header and data.
3. Computes SHA-1 hash of the full data.
4. If `write` is `True`, compresses and writes the object in `.git/objects/xx/yyyy...` based on hash.
5. Returns the hex SHA-1 string.

**Example Usage:**  
```python
blob_hash = hash_object(b'Hello World\n', 'blob')
print(blob_hash)  # e.g., 'e59ff97941044f85df5297e1c302d260b6f8b1a5'
```

---

### 5. `read_index()`

**Purpose:**  
Reads the Git index file and parses its entries into `IndexEntry` objects.

**Parameters:**  
None

**Returns:**  
- List of `IndexEntry` objects representing files staged for commit.

**Operation Steps:**  
1. Reads `.git/index` file data; returns empty list if not found.
2. Verifies SHA-1 checksum of index contents.
3. Parses header to confirm signature and version.
4. Iteratively unpacks each index entry and associated file path.
5. Returns the list of parsed entries.

**Example Usage:**  
```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### 6. `add(paths)`

**Purpose:**  
Adds files to the Git index by hashing their current content and updating the index entries.

**Parameters:**  
- `paths` (list of str): List of file paths to add to the index.

**Returns:**  
None

**Operation Steps:**  
1. Normalizes file paths to use forward slashes.
2. Reads current index entries excluding those matching the given paths.
3. For each path:
   - Reads file content and hashes it as a blob.
   - Retrieves file system stats.
   - Constructs a new `IndexEntry`.
4. Appends new entries and sorts all by path.
5. Writes updated entries back to the index file.

**Example Usage:**  
```python
add(['README.md', 'src/main.py'])
```

---

### 7. `diff()`

**Purpose:**  
Displays a unified diff of all changed files between the index and the working copy.

**Parameters:**  
None

**Returns:**  
None (prints diff output)

**Operation Steps:**  
1. Calls `get_status()` to get changed files.
2. Reads index entries into a dictionary by path.
3. For each changed file:
   - Reads blob object data from index.
   - Reads current working copy file.
   - Uses `difflib.unified_diff()` to generate diff lines.
4. Prints the diff with file headers and separator lines between files.

**Example Usage:**  
```python
diff()
```

**Sample Output:**  
```
--- README.md (index)
+++ README.md (working copy)
@@ -1,3 +1,4 @@
-Line 1
+Line 1 modified
 Line 2
 Line 3
----------------------------------------------------------------------
```

---

### 8. `ls_files(details=False)`

**Purpose:**  
Lists the files recorded in the Git index, optionally showing details like mode, SHA-1, and stage number.

**Parameters:**  
- `details` (bool): If `True`, print detailed info per file.

**Returns:**  
None (prints output)

**Operation Steps:**  
- Iterates over index entries.
- Prints file paths or detailed info depending on `details`.

**Example Usage:**  
```python
ls_files()            # simple list of paths
ls_files(details=True)  # detailed list with mode and SHA-1
```

---

## ASCII Diagram: Working Copy Status Flow

```
Working Directory
       |
       | read all files (excluding .git)
       v
  Working paths set
       |
       | compare with
       v
   Git Index entries
       |
       | classify files as
       v
+------------------------------+
| changed | new | deleted files |
+------------------------------+
       |
       | status() prints summary
       v
    User output
```

---

## Summary

This document covers the essential functions for checking the status of the working copy in the `pygit` system. It explains how files are compared against the Git index to detect changes, additions, and deletions, how these are reported, and how diffs can be generated to inspect changes in detail. These functions form the backbone of the `git status` and `git diff` commands, providing users with critical feedback about their local changes within the repository.