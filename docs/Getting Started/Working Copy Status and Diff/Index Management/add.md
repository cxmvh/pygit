# add.md - Adding Files to the Git Index

## Overview

This document covers the functionality and usage of adding files to the Git index within the `pygit` project. The Git index (or staging area) is an intermediate space where changes to files are prepared before committing. This process involves hashing file contents as Git blob objects, creating index entries, and updating the index file.

The `add.md` file is part of the **Working Copy Status and Diff > Index Management** section in the overall documentation tree. It complements other index-related documentation such as `read_index.md` and `write_index.md`, focusing specifically on how files are staged (added) to the index. The ability to add files correctly is essential for preparing changes to be committed and pushed to a remote repository.

---

## Important Functions

### 1. `add(paths)`

#### Purpose

The `add` function stages one or more files by adding their current contents to the Git index. It processes each specified file path, hashes its contents into a Git blob object, creates a corresponding index entry, and writes the updated list of entries back to the index file.

This function is crucial for updating the index with new or modified files, enabling subsequent commit operations to include these changes.

#### Parameters

- `paths`: A list of file paths (strings) to be added to the index. Paths are normalized to use forward slashes (`/`).

#### Operation Steps

1. Normalize all input paths to use forward slashes (`/`).
2. Read the current index entries using `read_index()`.
3. Filter out any existing entries matching the paths to be added, ensuring no duplicates.
4. For each path:
   - Read the file content as bytes.
   - Hash the content as a `blob` object using `hash_object`.
   - Gather file metadata via `os.stat`.
   - Create a new `IndexEntry` with timestamps, device info, mode, user/group IDs, file size, SHA-1 hash, flags (path length), and the path.
5. Append the new entries to the filtered existing entries.
6. Sort all entries by path.
7. Write the updated entries back to the index file with `write_index()`.

#### Example Usage

```python
from pygit import add

# Add single file
add(['README.md'])

# Add multiple files
add(['src/main.py', 'docs/manual.txt'])
```

#### Notes

- The function assumes the files exist and are readable.
- The `flags` field in `IndexEntry` stores the path length; paths longer than 4095 bytes are not supported.
- The index file is updated atomically after processing all files.

---

## Related Functions (for Context)

### `read_index()`

Reads the index file and returns a list of `IndexEntry` objects representing all staged files.

### `hash_object(data, obj_type, write=True)`

Hashes given data as a Git object of the specified type (`blob`, `tree`, `commit`), optionally writing it to the object store.

### `write_index(entries)`

Writes a list of `IndexEntry` objects back to the Git index file, ensuring proper format and checksum.

### `read_file(path)`

Utility function to read the contents of a file as bytes.

---

## ASCII Diagram: Adding Files to Git Index Flow

```
+-----------------+
|  User Calls     |
|  add(paths)     |
+--------+--------+
         |
         v
+----------------------+
| Normalize file paths  |
+----------------------+
         |
         v
+---------------------------+
| Read current index entries |
|   (read_index)            |
+---------------------------+
         |
         v
+---------------------------------------------+
| Remove entries for paths being added         |
+---------------------------------------------+
         |
         v
+---------------------------------------------+
| For each path:                              |
|  - Read file content (read_file)            |
|  - Hash content as blob (hash_object)       |
|  - Get file metadata (os.stat)               |
|  - Create IndexEntry                         |
+---------------------------------------------+
         |
         v
+------------------------+
| Append new entries      |
| Sort by path           |
+------------------------+
         |
         v
+--------------------+
| Write updated index |
|  (write_index)     |
+--------------------+
```

---

## Summary

The `add` function is the key interface for staging files in the `pygit` implementation, bridging the working directory and the Git index. It ensures that file contents are properly hashed, metadata is recorded, and the index file accurately reflects the staged state before committing.

This process is fundamental in Git workflows, enabling users to prepare and review changes prior to commits and pushes.

---

# Additional Usage Example with Context

```python
import pygit

# Stage all Python source files in current directory
pygit.add(['file1.py', 'file2.py'])

# Check status to confirm files are staged
pygit.status()

# Commit staged changes
pygit.commit('Add initial Python source files')
```

This sequence demonstrates typical usage of adding files to the index, verifying their staged state, and committing.

---

# End of add.md Documentation Content