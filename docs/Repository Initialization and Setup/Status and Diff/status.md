# status.md - Showing status of working copy including changed, new, and deleted files

---

## Overview

The `status.md` document covers the functionality related to checking and displaying the status of the working copy in a Git repository. It provides detailed insight into how the working copy is compared to the Git index to identify files that have been changed, added, or deleted. This file is part of the broader "Status and Diff" section within the "Index and Working Copy Management" area of the documentation. The status functionality is critical for users to understand what changes exist before committing them, enabling informed version control operations.

The key functions documented here include `get_status()` for retrieving the current status data and `status()` for printing that status in a human-readable format. These functions leverage core utilities such as reading the Git index, hashing file contents, and comparing working directory files. Other related functions like `add()` and `diff()` work in tandem to manage and display changes.

---

## Functions

### 1. `get_status()`

**Purpose:**  
Determines the status of the working copy by comparing the files present in the current directory (excluding `.git`) against the files recorded in the Git index. It returns three sorted lists of file paths categorized as changed, new, or deleted files.

**Signature:**  
```python
def get_status()
```

**Parameters:**  
None.

**Returns:**  
A tuple of three lists `(changed_paths, new_paths, deleted_paths)`:
- `changed_paths`: Files present both in the working directory and index but with different content.
- `new_paths`: Files present in the working directory but not in the index.
- `deleted_paths`: Files present in the index but missing from the working directory.

**Operation Steps:**

1. Traverse the working directory recursively, skipping the `.git` directory, to collect all file paths.
2. Normalize paths to use forward slashes and strip leading `./`.
3. Read the Git index to obtain entries representing tracked files.
4. Construct a dictionary mapping file paths to their index entries.
5. Identify changed files by comparing the SHA-1 hash of the file contents in the working directory with the SHA-1 stored in the index.
6. Identify new files as those present in the working directory but absent in the index.
7. Identify deleted files as those present in the index but missing from the working directory.
8. Return sorted lists of changed, new, and deleted paths.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 2. `status()`

**Purpose:**  
Prints a human-readable summary of the working copy status, listing changed, new, and deleted files.

**Signature:**  
```python
def status()
```

**Parameters:**  
None.

**Operation Steps:**

1. Calls `get_status()` to retrieve the lists of changed, new, and deleted files.
2. Prints each category with a heading if there are any files in that category.
3. Lists the file paths under each category indented for readability.

**Example Usage:**

```python
status()
```

**Sample Output:**

```
changed files:
    src/main.py
    README.md
new files:
    docs/usage.md
deleted files:
    old_script.py
```

---

### 3. `read_file(path)`

**Purpose:**  
Utility function to read the contents of a file as bytes.

**Signature:**  
```python
def read_file(path)
```

**Parameters:**  
- `path` (str): File path to read.

**Returns:**  
File contents as bytes.

**Example Usage:**

```python
data = read_file('README.md')
print(data.decode())
```

---

### 4. `hash_object(data, obj_type, write=True)`

**Purpose:**  
Computes the SHA-1 hash of given data as an object of a specified Git type (e.g., blob), optionally writing the object to the Git object store.

**Signature:**  
```python
def hash_object(data, obj_type, write=True)
```

**Parameters:**  
- `data` (bytes): Data to hash.
- `obj_type` (str): Type of Git object (e.g., `'blob'`).
- `write` (bool): Whether to write the object to the Git object store.

**Returns:**  
SHA-1 hash of the object as a hex string.

**Example Usage:**

```python
file_data = read_file('README.md')
sha1 = hash_object(file_data, 'blob')
print('SHA-1:', sha1)
```

---

### 5. `read_index()`

**Purpose:**  
Reads the Git index file and returns a list of `IndexEntry` objects representing the tracked files.

**Signature:**  
```python
def read_index()
```

**Returns:**  
List of `IndexEntry` objects.

**Operation Details:**

- Reads `.git/index`.
- Verifies SHA-1 checksum of index data.
- Parses entries with file metadata and SHA-1 hashes.

**Example Usage:**

```python
entries = read_index()
for entry in entries:
    print(entry.path, entry.sha1.hex())
```

---

### 6. `add(paths)`

**Purpose:**  
Adds the specified file paths to the Git index, updating it with their current content.

**Signature:**  
```python
def add(paths)
```

**Parameters:**  
- `paths` (list of str): File paths to add to the index.

**Operation Steps:**

1. Normalize path separators.
2. Read current index entries.
3. Remove entries for the specified paths to avoid duplicates.
4. For each path:
    - Read file content.
    - Compute SHA-1 hash.
    - Gather file stat info.
    - Create new `IndexEntry` and append.
5. Sort entries by path and write back the index.

**Example Usage:**

```python
add(['README.md', 'src/main.py'])
```

---

### 7. `diff()`

**Purpose:**  
Displays the unified diff of files that have changed between the index and the working copy.

**Signature:**  
```python
def diff()
```

**Operation Steps:**

1. Get list of changed files using `get_status()`.
2. For each changed file:
    - Read blob data from Git object store.
    - Read current working copy file.
    - Use `difflib.unified_diff` to compute diff.
    - Print the diff with headers indicating index and working copy versions.
3. Separate diffs with a line of dashes.

**Example Usage:**

```python
diff()
```

---

## ASCII Diagram: Status Check Workflow

```
+-------------------+        +------------------+        +------------------+
| Working Directory  |        | Git Index        |        | Status Result    |
| (files on disk)    |        | (index entries)  |        | - Changed files  |
+-------------------+        +------------------+        | - New files      |
          |                          |                   | - Deleted files  |
          |                          |                   +------------------+
          |                          |
          |                          |
          v                          v
   Collect all files         Read index entries
          |                          |
          +------------+-------------+
                       |
        Compare files on disk with index entries
                       |
                       v
          Categorize files as changed, new, or deleted
```

---

## Summary

This documentation explains the core mechanisms behind the `status` feature of the Git implementation, detailing how file changes are detected and reported. The combination of `get_status()` and `status()` functions enables inspection of the working copy state, a foundational operation for any version control workflow. The use of helper functions like `read_file`, `hash_object`, and `read_index` illustrates the integration of file system interaction, object hashing, and index management to achieve accurate status reporting.