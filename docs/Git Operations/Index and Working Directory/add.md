# add.md — Adding Files to the Git Index

## Overview

This document explains how files are added to the Git index (also known as the staging area) within the `pygit` project. The Git index tracks which files are staged for the next commit by recording their metadata and hashed contents. The `add.md` file fits into the "Working Copy Status and Diff" section under the "Index Management" subsection of the overall documentation tree, complementing other index-related documentation such as listing files (`ls_files.md`), reading (`read_index.md`), and writing (`write_index.md`) the index.

Adding files to the index is a crucial step in the Git workflow, preparing changes to be committed. This documentation focuses primarily on the `add(paths)` function, which efficiently updates the index with one or more file paths, along with explanations of its supporting functions and data structures.

---

## Function Documentation

### `add(paths)`

#### Purpose

Add one or more file paths to the Git index, updating the index entries with the current file contents and metadata. This function stages the specified files, preparing them for the next commit.

#### Parameters

- `paths` (list of str): List of file paths to add to the index. Paths are normalized to use forward slashes `/`.

#### Detailed Operation

1. Normalize all input paths by replacing backslashes (`\`) with forward slashes (`/`).
2. Read the current index entries by calling `read_index()`.
3. Filter out any existing entries from the index that match the paths to be added, to avoid duplicates.
4. For each path to add:
   - Read the file contents as bytes.
   - Compute the SHA-1 hash of the file contents (as a `blob` Git object) using `hash_object()`.
   - Retrieve file system metadata (`stat`) such as creation/modification times, device, inode, mode, user/group IDs, and file size.
   - Create a new `IndexEntry` with the metadata, computed SHA-1, and path.
5. Append these new entries to the filtered list.
6. Sort all entries by their file path lexicographically.
7. Write the updated list back to the Git index file using `write_index()`.

This process ensures that the index accurately reflects the current state of the staged files.

#### Example Usage

```python
# Stage multiple files
files_to_add = ['README.md', 'src/main.py', 'docs/usage.md']
add(files_to_add)
print("Files added to index and staged for commit.")
```

---

### Supporting Functions and Concepts

#### `read_index()`

Reads the Git index file and returns a list of `IndexEntry` objects representing the staged files.

- Parses the binary index file format.
- Validates the index signature and checksum.
- Decodes metadata and file paths for each entry.

#### `hash_object(data, obj_type, write=True)`

Generates a SHA-1 hash for the given data of a specified Git object type (`blob`, `tree`, `commit`, etc.).

- Creates the Git object header.
- Optionally writes the compressed object to the `.git/objects` directory.
- Returns the hex SHA-1 hash string.

#### `write_index(entries)`

Writes a list of `IndexEntry` objects back to the Git index file (`.git/index`).

- Packs entries with their metadata and paths into the correct binary format.
- Adds a checksum for index integrity.
- Overwrites the existing index file with the new data.

---

## Data Structure: `IndexEntry`

Each entry in the Git index is represented by an `IndexEntry` object containing:

- `ctime_s`, `ctime_n`: Creation time seconds and nanoseconds
- `mtime_s`, `mtime_n`: Modification time seconds and nanoseconds
- `dev`: Device number
- `ino`: Inode number
- `mode`: File mode (permissions and type)
- `uid`: User ID of owner
- `gid`: Group ID of owner
- `size`: File size in bytes
- `sha1`: 20-byte SHA-1 hash of the file contents
- `flags`: Flags including the path length
- `path`: File path (string)

---

## Illustrative ASCII Diagram: Git Index Update Flow for `add(paths)`

```
+------------------+
|   Input paths    |  ---> Normalize paths (replace '\' with '/')
+------------------+
          |
          v
+------------------+
|  read_index()    |  ---> Read existing index entries
+------------------+
          |
          v
+-----------------------------------------+
| Filter out entries matching input paths |  ---> Remove old entries for these paths
+-----------------------------------------+
          |
          v
+-------------------------------------+
| For each path:                      |
|  - read_file(path)                  |
|  - hash_object(data, 'blob')        |
|  - os.stat(path)                    |
|  - create IndexEntry                |
+-------------------------------------+
          |
          v
+----------------------+
| Append new entries    |
+----------------------+
          |
          v
+-------------------------+
| Sort entries by path    |
+-------------------------+
          |
          v
+----------------------+
| write_index(entries)  |  ---> Write updated index back to disk
+----------------------+
```

---

## Additional Notes

- The `add()` function supports adding multiple files at once, which is efficient when staging many changes.
- This implementation currently assumes flat file paths without nesting subdirectories (no recursive adding of directory contents).
- It relies on underlying functions like `hash_object()` to store objects in the Git object database correctly.
- The index file format and checksum are strictly validated to maintain repository consistency.

---

## Summary

The `add.md` documentation details the process of adding files to the Git index in `pygit` via the `add(paths)` function. This operation stages file changes by updating the index with file metadata and SHA-1 hashes of their contents. Understanding this process is essential for workflows involving commits and pushes, as the index reflects the set of changes that will be committed to the repository. The document also references related index manipulation functions to provide a comprehensive understanding of index management.