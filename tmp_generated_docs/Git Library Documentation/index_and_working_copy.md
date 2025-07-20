---
sidebar_position: 3
---

# Git Index and Working Copy Management

## Overview

This document provides a comprehensive reference for handling the Git index and the working copy within the pygit library. It explains the Git index format, procedures for reading and writing index entries, and methods for adding files to the index. The documentation also covers listing files tracked by the index via commands like `ls_files`. Additionally, it details managing the working copy's status, detecting file changes, and displaying summarized status reports. This file plays a crucial role in bridging the repository's stored state and the current working directory, enabling efficient synchronization and change tracking.

---

## `read_index`

### Purpose

Reads and parses the Git index file, loading its entries into memory. The index contains metadata about files staged for the next commit, such as their paths, SHA-1 hashes, and stat information.

### Parameters

- **index_path** (`str`): Path to the Git index file (usually `.git/index`).

### Operation

1. Opens the index file in binary mode.
2. Validates the header signature (`'DIRC'`) and version number.
3. Reads the number of index entries.
4. Iteratively reads each index entry, which includes:
    - File metadata (timestamps, device/inode numbers, mode, UID/GID, size).
    - SHA-1 hash of the file contents.
    - Flags and file path.
5. Stores all entries in an internal data structure for further operations.

### Example

```python
index_entries = read_index('.git/index')
for entry in index_entries:
    print(f"Tracked file: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

## `write_index`

### Purpose

Writes the in-memory index entries back to the Git index file, updating the persistent record of the staged files.

### Parameters

- **index_path** (`str`): Path where the index file will be written.
- **entries** (`list`): List of index entry objects to write.

### Operation

1. Constructs the index header including signature and version.
2. Writes the number of entries.
3. Serializes each index entry with appropriate metadata and path.
4. Computes and appends a SHA-1 checksum of all written data for integrity.
5. Saves the serialized data atomically to the index file.

### Example

```python
write_index('.git/index', index_entries)
```

---

## `add`

### Purpose

Adds or updates files in the Git index, staging them for the next commit.

### Parameters

- **paths** (`list` of `str`): File paths to add to the index.
- **index_entries** (`list`): Current index entries, to be updated.

### Operation

1. For each file path:
    - Reads file content and computes its SHA-1 hash.
    - Collects file stat information (mode, timestamps, size).
    - Creates or updates the corresponding index entry.
2. Updates the index entries list with new or changed entries.
3. Calls `write_index` to persist changes.

### Example

```python
add(['README.md', 'src/main.py'], index_entries)
```

---

## `ls_files`

### Purpose

Lists the files currently tracked in the index.

### Parameters

- **index_entries** (`list`): The loaded index entries.

### Operation

- Iterates over all index entries and outputs file paths with optional metadata.

### Example

```python
for path in ls_files(index_entries):
    print(path)
```

---

## `get_status`

### Purpose

Determines the state of files in the working copy relative to the index and HEAD commit, identifying modifications, additions, deletions, and untracked files.

### Parameters

- **index_entries** (`list`): Current index entries.
- **head_tree** (`dict`): Tree object representing the HEAD commit's file state.
- **working_dir** (`str`): Path to the working directory.

### Operation

1. Compares working directory files against index entries:
    - Checks for modifications by comparing file stats and hashes.
2. Compares index entries against HEAD tree to detect staged changes.
3. Detects untracked files by scanning the working directory.
4. Compiles a status summary indicating file states (e.g., modified, staged, untracked).

### Example

```python
status = get_status(index_entries, head_tree, '/repo/working_copy')
print(status.summary())
```

---

## `diff`

### Purpose

Computes differences between two states, such as between the working copy and index, or index and HEAD commit.

### Parameters

- **old_tree** (`dict`): File tree representing the original state.
- **new_tree** (`dict`): File tree representing the changed state.

### Operation

1. Recursively compares file entries in the two trees.
2. Identifies added, deleted, and modified files.
3. Optionally generates line-by-line diffs for individual files.

### Example

```python
diffs = diff(head_tree, index_tree)
for diff_entry in diffs:
    print(diff_entry)
```

---

## `status`

### Purpose

Combines the functionality of `get_status` and `diff` to provide a user-friendly summary of the repository's current status relative to the index and HEAD commit.

### Parameters

- **working_dir** (`str`): Path to the working directory.
- **index_entries** (`list`): Current index entries.
- **head_tree** (`dict`): Tree object for the HEAD commit.

### Operation

1. Calls `get_status` to detect staged and unstaged changes.
2. Calls `diff` to generate detailed change information.
3. Formats the results into a concise status report similar to `git status`.

### Example

```python
print(status('/repo/working_copy', index_entries, head_tree))
```

---

## ASCII Diagram: Relationship Between HEAD, Index, and Working Copy

```
+-----------------+       +----------------+       +------------------+
|   HEAD Commit   |  <--> |     Index      |  <--> |   Working Copy   |
| (Last Commited) |       | (Staging Area) |       | (Unstaged Files) |
+-----------------+       +----------------+       +------------------+
        ^                          ^                          ^
        |                          |                          |
  Commit points to          Files added via              Files edited,
  a tree object             'add' command               created or deleted
```

This diagram illustrates how the HEAD commit, the index, and the working copy relate to each other in Git's workflow. The index acts as an intermediary staging area between the committed state and the current working directory.

---

This documentation provides a detailed reference for developers working with Git index and working copy management in the pygit library, enabling effective staging, tracking, and status reporting of file changes.