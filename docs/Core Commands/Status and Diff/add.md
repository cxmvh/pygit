# add.md - Adding Files to the Git Index and Index Management

---

## Overview

The `add.md` document details the process of adding files to the Git index, an essential step in preparing changes for commit in a Git repository implemented via the `pygit` library. This file explains how files from the working directory are hashed into Git objects (blobs), how index entries are created or updated, and how the index file (`.git/index`) is managed internally. It fits within the broader "Status and Diff" section of the documentation, complementing status reporting and diffing by describing how files are staged for commits.

Adding files to the index is a core operation that bridges the working copy and the Git object database. Understanding this process provides insight into Git's internal mechanisms for tracking file contents and changes efficiently.

---

## Function Documentation

### `add(paths)`

#### Purpose

The `add` function stages one or more files by adding their current contents to the Git index. This prepares the files to be included in the next commit. It updates the index entries for the specified paths, creating new entries or replacing existing ones.

#### Parameters

- `paths` (`list` of `str`): A list of file paths (relative to the repository root) to be added to the index. Paths are normalized to use forward slashes.

#### Operation Steps

1. Normalize all paths to use Unix-style forward slashes for consistency.
2. Read the current index entries using `read_index()`.
3. Filter out any existing entries in the index that have the same paths as those to be added (to avoid duplicates).
4. For each specified path:
   - Read the file contents from the working directory.
   - Compute the SHA-1 hash of the file content as a Git blob object using `hash_object`.
   - Retrieve the file's metadata (creation time, modification time, device, inode, mode, user/group IDs, and size) using `os.stat`.
   - Create a new `IndexEntry` object encapsulating the file metadata, computed SHA-1 hash, a flags field containing the path length, and the file path.
5. Append the new entries to the filtered list of existing entries.
6. Sort all entries by their file path lexicographically.
7. Write the updated list of entries back to the index file using `write_index()`.

#### Preconditions

- The file paths must exist in the working directory.
- The `.git` directory must be initialized and accessible.
- The `IndexEntry` class and `write_index` function must be implemented correctly.

#### Example Usage

```python
# Stage two files for the next commit
files_to_add = ['src/main.py', 'README.md']
add(files_to_add)
```

This call will read the contents of `src/main.py` and `README.md`, create blob objects for their contents, and update the index accordingly.

---

### Supporting Functions (Brief Overview)

The `add` function relies on several supporting functions and concepts, briefly described here for context:

- **`read_index()`**  
  Reads the `.git/index` file and returns a list of `IndexEntry` objects representing the current staging area.

- **`hash_object(data, obj_type, write=True)`**  
  Converts file data into a Git object of the specified type (`blob` for files), writes it to the object store, and returns the SHA-1 hash.

- **`write_index(entries)`**  
  Writes a list of `IndexEntry` objects back to the `.git/index` file, including proper header and checksum.

- **`IndexEntry`**  
  A data structure representing metadata and SHA-1 hash for a staged file.

---

## Index Entry Structure and Index File Layout

The Git index file stores metadata for each staged file, including timestamps, device and inode information, file mode, ownership, size, SHA-1 hash of the blob object, flags, and the file path.

```
+----------------------------+--------------------------------+
| Index File Header           | 12 bytes                      |
|  - Signature 'DIRC' (4)     | Version number (4)             |
|  - Number of entries (4)    |                               |
+----------------------------+--------------------------------+
| Index Entries (variable)                                   |
|  - Fixed-size metadata (62 bytes)                         |
|  - Null-terminated file path                              |
|  - Padding to 8-byte alignment                            |
+-----------------------------------------------------------+
| SHA-1 checksum of all previous bytes (20 bytes)          |
+-----------------------------------------------------------+
```

### ASCII Diagram of Index Entry Layout

```
+-------------------------------+
| ctime seconds         (4 bytes)|
| ctime nanoseconds     (4 bytes)|
| mtime seconds         (4 bytes)|
| mtime nanoseconds     (4 bytes)|
| dev                   (4 bytes)|
| ino                   (4 bytes)|
| mode                  (4 bytes)|
| uid                   (4 bytes)|
| gid                   (4 bytes)|
| file size             (4 bytes)|
| SHA-1 hash            (20 bytes)|
| flags                 (2 bytes)|
| path                  (variable)|
| null terminator        (1 byte) |
| padding to 8-byte boundary    |
+-------------------------------+
```

This structure allows Git to efficiently track file state and content integrity.

---

## Additional Notes

- The `add` operation only stages file contents and metadata; it does not modify the working directory or commit history.
- The index acts as the staging area, reflecting what will be included in the next commit.
- Large repositories rely on efficient hashing and indexing to speed up operations.
- The flags field in `IndexEntry` encodes the path length and stage number, supporting advanced Git features like merge conflicts.

---

# Summary

This document provides a detailed look at how files are added to the Git index within the `pygit` implementation, focusing on the `add(paths)` function. It outlines the full workflow from reading the working directory files to updating the binary index file with new or updated entries. Understanding this process is crucial for grasping Git's internal staging mechanisms and how changes are prepared for committing.