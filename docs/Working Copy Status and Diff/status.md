# status.md

# Working Copy Status

## Overview

This document details how the working copy’s status is determined and displayed, focusing on identifying changed, new, and deleted files in the repository. It is part of the "Working Copy Status and Diff" section, which provides insight into how the repository tracks modifications in the working area relative to the index and committed snapshots.

The status functionality plays a crucial role in helping users understand which files have been modified, added, or removed before committing changes. It integrates with core components such as the Git index, object storage, and file system operations.

---

## Functions

### `status()`

**Purpose:**  
Displays a summary of the working copy status by printing lists of changed, new, and deleted files.

**Parameters:**  
None.

**Operation:**  
1. Calls `get_status()` to retrieve three lists: changed files, new files, and deleted files.  
2. For each category, if any files are present, prints a heading and then lists the file paths indented for readability.

**Example Usage:**

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

### `get_status()`

**Purpose:**  
Computes and returns the status of the working copy as three sorted lists: changed files, new files, and deleted files.

**Parameters:**  
None.

**Returns:**  
`(changed_paths, new_paths, deleted_paths)` — each a sorted list of file paths as strings.

**Operation:**  
1. Walks through the working directory recursively, ignoring the `.git` directory.  
2. Collects all file paths relative to the repository root.  
3. Reads the Git index entries and maps them by their paths.  
4. Determines:
   - **Changed files:** Files present both in the working copy and index but with differing content hashes.  
   - **New files:** Files present in working copy but not in index.  
   - **Deleted files:** Files present in index but missing from working copy.  
5. Returns sorted lists of paths for these three categories.

**Step-by-Step:**

```
+--------------------+
|  Working directory  |
+--------------------+
          |
          v
+--------------------+       +------------------+
|  Get all file paths |       | Read index entries|
+--------------------+       +------------------+
          |                            |
          +------------+---------------+
                       |
                       v
        +---------------------------------------+
        | Compute sets: changed, new, deleted    |
        +---------------------------------------+
                       |
                       v
             Return three sorted lists
```

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `read_file(path)`

**Purpose:**  
Reads the raw bytes of a file from the working directory.

**Parameters:**  
- `path` (str): The relative path to the file.

**Returns:**  
Byte string of the file contents.

**Operation:**  
Opens the file in binary mode and reads all contents.

**Example Usage:**

```python
data = read_file("src/main.py")
print(data[:100])  # Print first 100 bytes of the file
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of an object of a specified type and optionally writes it to the Git object store.

**Parameters:**  
- `data` (bytes): The raw data of the object.  
- `obj_type` (str): The type of object (e.g., `'blob'`, `'tree'`, `'commit'`).  
- `write` (bool): Whether to write the object to the `.git/objects` directory.

**Returns:**  
Hexadecimal SHA-1 hash string of the object.

**Operation:**  
1. Constructs the object header: `<type> <length>`.  
2. Concatenates header, a null byte, and data.  
3. Hashes the resulting bytes with SHA-1.  
4. If `write` is True, compresses and writes the object to `.git/objects/<first_two_chars>/<remaining_chars>`.  
5. Returns the SHA-1 hash.

**Example Usage:**

```python
data = read_file("README.md")
sha1 = hash_object(data, "blob")
print("Object hash:", sha1)
```

---

### `read_index()`

**Purpose:**  
Reads the Git index file and parses its entries into a list of `IndexEntry` objects.

**Parameters:**  
None.

**Returns:**  
List of `IndexEntry` objects representing files staged in the index.

**Operation:**  
1. Attempts to read `.git/index`. Returns an empty list if not found.  
2. Validates the index checksum and header signature/version.  
3. Iteratively parses entries formatted as fixed-size metadata plus variable-length path strings.  
4. Returns list of entries with file metadata, SHA-1 hashes, and paths.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(f"{entry.path} -> {entry.sha1.hex()}")
```

---

## ASCII Diagram: Status Determination Flow

```
+---------------------------+
| Start: Working Directory  |
+---------------------------+
            |
            v
+-------------------+      +--------------------+
| List all files     |      | Read Git index file |
| in working copy    |      | (list of entries)   |
+-------------------+      +--------------------+
            |                      |
            +----------+-----------+
                       |
                       v
     +-----------------------------------+
     | Categorize files by comparing:    |
     | - Content hash of working copy    |
     |   vs. index entries                |
     +-----------------------------------+
                       |
           +-----------+------------+
           |           |            |
           v           v            v
      Changed       New files     Deleted files
     (modified)    (untracked)   (missing in work)
```

---

## Summary

The `status.md` documentation covers the key functions responsible for determining and displaying the working copy status, focusing on:

- Enumerating all files in the working directory.
- Comparing file contents with the index.
- Categorizing files into changed, new, or deleted.
- Presenting these results to the user in a concise format.

This functionality enables users to be aware of modifications before committing, forming an essential part of the version control workflow.