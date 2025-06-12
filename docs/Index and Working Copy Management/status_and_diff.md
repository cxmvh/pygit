# status_and_diff.md

---

## Overview

This document details the functions responsible for obtaining the working copy status and displaying diffs between the Git index and the working copy in the pygit project. It fits within the broader "Index and Working Copy Management" section of the documentation tree, complementing files like `status.md` and `diff.md` by providing the key utilities to detect changes, new files, and deletions, and to visualize differences at the file content level. These functions are crucial for tracking modifications before staging or committing changes, thus forming an essential part of the Git workflow implemented by pygit.

---

## Function Documentation

### `get_status()`

**Purpose:**  
`get_status` scans the working directory (excluding `.git`) and compares the current files to the Git index entries. It returns three sorted lists representing files that have changed content, new files not yet indexed, and deleted files that are in the index but missing in the working copy.

**Parameters:**  
- None

**Returns:**  
- `changed_paths` (list of str): Files modified since last index update.  
- `new_paths` (list of str): Files present in working directory but not in the index.  
- `deleted_paths` (list of str): Files recorded in the index but missing in the working directory.

**Operation Details:**  
1. Traverse the working directory recursively, ignoring `.git` directory, to collect all file paths.  
2. Normalize paths to use forward slashes and remove leading `./`.  
3. Read the Git index entries and map them by path.  
4. Identify changed files by hashing current file contents and comparing against the stored SHA-1 in the index.  
5. Identify new files as those present in working directory but not in the index.  
6. Identify deleted files as those present in the index but missing in the working directory.  
7. Return sorted lists of each category.

**Example Usage:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `diff()`

**Purpose:**  
`diff` displays unified diffs between the files in the Git index and the working copy for files detected as changed. This helps users visualize line-by-line differences before staging or committing.

**Parameters:**  
- None

**Operation Details:**  
1. Call `get_status()` to obtain the list of changed files.  
2. Read the index entries and map by path to get the SHA-1 for each changed file.  
3. For each changed file:  
   - Read the blob object from the object store using the SHA-1 from the index.  
   - Decode the blob to lines representing the index version of the file.  
   - Read the file contents from the working copy and decode to lines.  
   - Use `difflib.unified_diff` to generate a unified diff between the index and working copy versions.  
4. Print the diff output with headers indicating `(index)` and `(working copy)` versions.  
5. Separate diffs for multiple files with a line of dashes.

**Example Usage:**

```python
diff()
```

**Sample Output:**

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

---

### `status()`

**Purpose:**  
`status` provides a user-friendly printout of the working copy state — listing changed, new, and deleted files based on the current index and working directory.

**Parameters:**  
- None

**Operation Details:**  
1. Calls `get_status()` to retrieve changed, new, and deleted file lists.  
2. Prints sections for changed, new, and deleted files only if those lists are non-empty.  
3. Each file path is printed indented for readability.

**Example Usage:**

```python
status()
```

**Sample Output:**

```
changed files:
    README.md
new files:
    new_script.py
deleted files:
    old_module.py
```

---

## ASCII Diagram: Workflow of Status and Diff Functions

```
+--------------------+
|   Working Directory |
|   (files on disk)   |
+----------+---------+
           |
           | Scan files (excluding .git)
           v
+--------------------+         +-----------------+
|   Git Index File    | <-----> |  Index Entries  |
|  (.git/index)       |         |  (path, SHA1)   |
+--------------------+         +-----------------+
           |                           ^
           |                           |
           | Compare file hashes      | Stored SHA-1
           v                           |
+--------------------+         +-----------------+
|  get_status()       |-------->| Identify changed|
|  - Collect paths    |         | new, deleted     |
|  - Compare hashes   |         +-----------------+
+--------------------+
           |
           | Changed files list
           v
+--------------------+
|      diff()         |
|  - Read index blobs |
|  - Read working    |
|    files           |
|  - Generate diff   |
+--------------------+
```

---

## Related Functions Overview

While this document focuses on status and diff display, it relies on several underlying functions to perform its tasks:

- `read_index()`: Reads and parses the Git index file, returning index entries.  
- `read_object(sha1_prefix)`: Reads Git objects (e.g., blobs) from the object store.  
- `hash_object(data, obj_type, write=False)`: Computes the SHA-1 hash of file contents without writing.  
- `read_file(path)`: Reads file contents from the working directory.

These functions are detailed in other documentation files such as `index.md`, `objects.md`, and `status.md`.

---

# End of status_and_diff.md Documentation Content