---
sidebar_position: 1
---

# Status and Index Management

## Overview

This document provides unified reference material for status and index management within the repository workflow. It covers the implementation and usage of the `status` command, which determines changed, new, and deleted files by comparing the working directory, the index, and the repository state. The document also details key supporting functions such as `get_status`, `read_index`, `hash_object`, and the `add` function. Additionally, it explains the structure and handling of index entries (`IndexEntry`), as well as reading from and writing to the index file. This file belongs to the *Change Workflows* section of the documentation tree, complementing related operations such as diffing, committing, and pushing changes.

---

## Functions and Concepts

### `get_status`

**Purpose:**  
`get_status` is the primary function responsible for determining the status of files in the working directory relative to the Git index and repository. It identifies files that are untracked, modified, or deleted by checking file metadata and content hashes.

**Parameters:**  
- `repo_path` (str): Path to the repository root directory.
- `index` (list of `IndexEntry`): Current list of index entries representing tracked files.
- `working_dir_files` (list of str): List of file paths currently present in the working directory.

**Operation:**  
1. Reads the current index entries.
2. Scans the working directory files.
3. Compares each working directory file with the corresponding index entry (if any):
   - Checks for new files not in the index.
   - Detects modified files by comparing stored hashes or file stats.
4. Identifies deleted files by detecting index entries with no corresponding working directory file.
5. Returns a summary data structure categorizing files as new, modified, deleted, or unchanged.

**Example Usage:**

```python
index_entries = read_index(repo_path)
working_files = list_working_directory_files(repo_path)
status = get_status(repo_path, index_entries, working_files)

print("New files:", status['new'])
print("Modified files:", status['modified'])
print("Deleted files:", status['deleted'])
```

---

### `read_index`

**Purpose:**  
Reads the Git index file (`.git/index`) and parses it into a list of `IndexEntry` objects representing the current staging area state.

**Parameters:**  
- `repo_path` (str): Path to the repository root directory.

**Operation:**  
1. Opens and reads the binary index file.
2. Validates the index header and checksum.
3. Parses entries sequentially into `IndexEntry` instances, extracting file mode, SHA-1 hash, timestamps, and file path.
4. Returns a list of `IndexEntry` objects.

**Example Usage:**

```python
index_entries = read_index("/path/to/repo")
for entry in index_entries:
    print(f"Tracked file: {entry.path} with SHA {entry.sha.hex()}")
```

---

### `hash_object`

**Purpose:**  
Computes the SHA-1 hash of a file's contents and optionally writes the object to the object database, facilitating object storage and identification.

**Parameters:**  
- `file_path` (str): Path to the file to hash.
- `write` (bool): Whether to write the object to the object database (default: False).

**Operation:**  
1. Reads the file content.
2. Prepends the appropriate Git object header (`blob <size>\0`).
3. Computes the SHA-1 hash of the header plus content.
4. If `write` is True, compresses and stores the object in the `.git/objects` directory.
5. Returns the SHA-1 hash bytes.

**Example Usage:**

```python
sha = hash_object("/path/to/file.txt", write=True)
print(f"Object stored with SHA: {sha.hex()}")
```

---

### `add`

**Purpose:**  
Stages a file by adding or updating its entry in the Git index, preparing it for the next commit.

**Parameters:**  
- `repo_path` (str): Path to the repository root directory.
- `file_path` (str): Relative path of the file to add.

**Operation:**  
1. Reads the file contents and computes its SHA-1 hash, writing the object to the database.
2. Creates or updates an `IndexEntry` for the file, including file mode, last modified time, and SHA.
3. Reads the existing index entries.
4. Updates the index list with the new or modified entry.
5. Writes the updated list back to the index file, updating the checksum.

**Example Usage:**

```python
add("/path/to/repo", "src/main.py")
print("File 'src/main.py' added to staging area.")
```

---

### `IndexEntry` Structure

An `IndexEntry` represents a single tracked file in the Git index. It contains metadata and object information used for status calculation and commit preparation.

| Field         | Description                                         |
|---------------|-----------------------------------------------------|
| `ctime`       | Creation time of the file (seconds and nanoseconds) |
| `mtime`       | Last modification time (seconds and nanoseconds)    |
| `dev`         | Device number                                        |
| `ino`         | Inode number                                        |
| `mode`        | File mode (permissions and type)                    |
| `uid`         | User ID of file owner                               |
| `gid`         | Group ID of file owner                              |
| `size`        | Size of the file in bytes                           |
| `sha`         | SHA-1 hash of the file content                      |
| `flags`       | Bit flags representing path length and stage       |
| `path`        | Relative file path as a string                      |

---

### Index File Reading and Writing

The index file is a binary file storing `IndexEntry` records along with a header and checksum. Proper reading and writing involve:

- Parsing the 12-byte header (`signature`, `version`, `entry count`).
- Reading each `IndexEntry` with correct padding to align entries to an 8-byte boundary.
- Computing and validating a SHA-1 checksum of all preceding bytes for integrity.
- Writing follows the reverse: serialize entries with proper padding, write the header, and append the checksum.

---

## ASCII Diagram: Index Entry Layout

```
+---------------------+
|   4 bytes: ctime_s  |
+---------------------+
|   4 bytes: ctime_ns |
+---------------------+
|   4 bytes: mtime_s  |
+---------------------+
|   4 bytes: mtime_ns |
+---------------------+
|   4 bytes: dev      |
+---------------------+
|   4 bytes: ino      |
+---------------------+
|   4 bytes: mode     |
+---------------------+
|   4 bytes: uid      |
+---------------------+
|   4 bytes: gid      |
+---------------------+
|   4 bytes: size     |
+---------------------+
|  20 bytes: sha-1    |
+---------------------+
|   2 bytes: flags    |
+---------------------+
|variable: file path  |
+---------------------+
| padding to 8-byte   |
| alignment           |
+---------------------+
```

---

This documentation file equips developers with a clear understanding of how status and index management is implemented, enabling effective interaction with the staging area and file state tracking in the repository workflow. For more detailed information on related workflows, see [diff.md](diff.md), [commit.md](commit.md), and [push.md](push.md) within the *Change Workflows* section.