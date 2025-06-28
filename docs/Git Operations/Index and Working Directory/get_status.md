# get_status.md

# Determining Changed, New, and Deleted Files in the Working Tree

---

## Overview

The `get_status.md` document provides detailed technical documentation for the `get_status` functionality within the pygit project. This file is part of the **Git Index and Working Copy** section under the broader **Working Copy Status and Diff** category, which focuses on tools for inspecting the working copy and differences with the Git index.

The `get_status` function is fundamental for identifying the state of files in the working directory relative to the Git index. Specifically, it detects files that have been changed, newly created, or deleted since the last commit or indexing. This status detection is essential for operations like staging changes (`add`), committing, and pushing, as well as for providing feedback to users on what has been modified in their working directory.

By integrating with index reading, object hashing, and file system inspection, the `get_status` function helps pygit mimic the behavior of `git status`, enabling a lightweight and transparent status reporting mechanism.

---

## Function Documentation

### `get_status()`

**Purpose:**  
Determine the status of the working copy by identifying files that have changed, are new, or have been deleted compared to the current Git index.

**Signature:**  
```python
def get_status():
```

**Returns:**  
A tuple of three lists:
- `changed_paths`: Sorted list of file paths that exist both in the working directory and index but whose contents differ.
- `new_paths`: Sorted list of file paths that exist in the working directory but are not yet tracked in the index.
- `deleted_paths`: Sorted list of file paths that exist in the index but no longer present in the working directory.

**Detailed Description:**  
1. **Collect all working directory files:**  
   Recursively walk through the current directory (`.`), excluding the `.git` directory, to gather all file paths in the working directory.

2. **Normalize paths:**  
   Paths are normalized to use forward slashes and are stripped of leading `./` for consistency with index entries.

3. **Read the Git index:**  
   Load the current index entries using the `read_index()` function, which returns a list of `IndexEntry` objects. These entries represent files currently tracked by Git.

4. **Build lookup for index entries:**  
   Create a dictionary mapping file paths to their respective `IndexEntry` objects for quick reference.

5. **Determine changed files:**  
   For files present both in the working directory and index, compute the SHA-1 hash of the current file contents (without writing to object store) and compare it to the SHA-1 stored in the index entry. If they differ, the file is marked as changed.

6. **Identify new files:**  
   Files found in the working directory but missing from the index are considered new.

7. **Identify deleted files:**  
   Files present in the index but missing from the working directory are considered deleted.

8. **Return sorted lists:**  
   The results are returned as sorted lists for predictable and consistent output.

**Usage Example:**

```python
changed, new, deleted = get_status()

print("Changed files:")
for path in changed:
    print("  ", path)

print("New files:")
for path in new:
    print("  ", path)

print("Deleted files:")
for path in deleted:
    print("  ", path)
```

**Example Output:**

```
Changed files:
   README.md
   src/main.py
New files:
   docs/usage.md
Deleted files:
   old_notes.txt
```

---

## Supporting Concepts and Dependencies

The `get_status` function relies on several other core functions and concepts within pygit:

- **`read_index()`**: Reads the `.git/index` file and returns a list of `IndexEntry` objects representing tracked files.
- **`hash_object(data, obj_type, write=False)`**: Computes the SHA-1 hash of the file content. When `write=False`, it does not write the object to the Git object database, used here only for comparison.
- **Filesystem traversal**: Uses `os.walk()` to traverse the working directory, excluding `.git` for performance and correctness.

---

## ASCII Diagram: Status Determination Flow

```
+------------------------+
| Start: Current Directory|
+-----------+------------+
            |
            v
+------------------------+
| Collect all file paths  |
| (exclude .git)          |
+-----------+------------+
            |
            v
+------------------------+
| Read Git index entries  |
+-----------+------------+
            |
            v
+------------------------+
| Compare working files   |
| with index entries:     |
|                        |
| - If in both: hash and  |
|   compare contents      |
| - If in working only:   |
|   mark new              |
| - If in index only:     |
|   mark deleted          |
+-----------+------------+
            |
            v
+------------------------+
| Return three lists:     |
| changed, new, deleted   |
+------------------------+
```

---

## Additional Notes

- The function currently treats file paths in a normalized manner, using forward slashes, which is important for cross-platform compatibility.
- This status information is typically consumed by higher-level commands such as `status()`, which formats and prints the status to the user, or by `add()` to stage files.
- The accuracy of `get_status()` depends on the integrity of the index and the correctness of the hashing function.
- Large repositories or directories with many files may experience performance impacts due to filesystem traversal and hashing.

---

## Related Functions in Documentation Tree

- **`status()`**: Uses `get_status()` to display human-readable status output.
- **`diff()`**: Uses `get_status()` to identify changed files and show diffs.
- **`add(paths)`**: Adds new or changed files to the index based on paths.
- **`read_index()`** / **`write_index()`**: Manage reading and writing the Git index.
- **`hash_object()`**: Compute SHA-1 hashes for objects.
- **`read_file()`** / **`write_file()`**: File I/O utilities.

---

This documentation provides a comprehensive reference for developers and users working with pygit's status functionality, enabling effective tracking of file changes in the working directory.