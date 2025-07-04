# Index and Working Copy Management

## Overview

This document details the functions responsible for managing the Git index and working copy within the pygit system. It covers how to read and write the index file, list files tracked by Git, stage files for commit, check the status of the working copy, and generate diffs for changes. These functions are foundational for tracking changes between the working directory and the repository’s committed state. This file fits within the broader "Index and Working Copy Management" section of the documentation tree, complementing other areas such as commit creation and object management.

---

## Function Documentation

### `read_index(index_path)`

**Purpose:**  
Reads the Git index file from the specified path and parses its contents into an in-memory data structure representing the current staging area.

**Parameters:**  
- `index_path` (str): The file path to the Git index (usually `.git/index`).

**Operation:**  
1. Opens the index file in binary mode.  
2. Reads and verifies the header signature and version.  
3. Parses the number of index entries.  
4. Iteratively reads each entry, extracting information such as file path, stat data, and SHA-1 object id.  
5. Constructs a list or dictionary representing all staged files and their metadata.

**Example Usage:**

```python
index_entries = read_index('.git/index')
for entry in index_entries:
    print(f"File: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### `write_index(index_path, index_entries)`

**Purpose:**  
Writes the in-memory index entries back to the Git index file, updating the staging area with new or modified entries.

**Parameters:**  
- `index_path` (str): The file path to write the Git index file.  
- `index_entries` (list): A list of index entry objects to serialize and write.

**Operation:**  
1. Serializes the header including signature and version.  
2. Writes the count of entries.  
3. Iteratively serializes each index entry, including file metadata and object SHA-1.  
4. Writes the complete binary content to the index file, ensuring atomic update.

**Example Usage:**

```python
# Assume 'index_entries' is updated or new entries list
write_index('.git/index', index_entries)
```

---

### `ls_files(repo_path, cached_only=True)`

**Purpose:**  
Lists files in the repository that are currently tracked in the index. Optionally filters by cached (staged) files only or includes untracked files.

**Parameters:**  
- `repo_path` (str): The root path of the repository.  
- `cached_only` (bool): If `True`, list only files staged in the index; if `False`, list files in the working directory as well.

**Operation:**  
1. Reads the index to obtain tracked files.  
2. If `cached_only` is `False`, scans the working directory for untracked files.  
3. Returns a list of file paths relative to the repository root.

**Example Usage:**

```python
tracked_files = ls_files('/path/to/repo')
print("Tracked files:", tracked_files)
```

---

### `stage_files(repo_path, file_paths)`

**Purpose:**  
Stages specified files by adding or updating their entries in the index, preparing them for commit.

**Parameters:**  
- `repo_path` (str): The root path of the repository.  
- `file_paths` (list of str): List of file paths (relative to repo root) to stage.

**Operation:**  
1. Reads the current index.  
2. For each file path, reads the file content and creates a blob object.  
3. Updates the index entry for each file with new SHA-1 and metadata.  
4. Writes the updated index back to disk.

**Example Usage:**

```python
files_to_stage = ['README.md', 'src/main.py']
stage_files('/path/to/repo', files_to_stage)
```

---

### `working_copy_status(repo_path)`

**Purpose:**  
Checks the status of files in the working copy relative to the index and HEAD commit, indicating modifications, additions, deletions, and untracked files.

**Parameters:**  
- `repo_path` (str): The root path of the repository.

**Operation:**  
1. Reads the index and HEAD tree to determine tracked files and their committed states.  
2. Compares working directory files to index entries to detect modifications or untracked files.  
3. Returns a structured status report detailing files and their status (e.g., modified, staged, untracked).

**Example Usage:**

```python
status_report = working_copy_status('/path/to/repo')
for file, status in status_report.items():
    print(f"{file}: {status}")
```

---

### `diff_working_copy(repo_path, file_path=None)`

**Purpose:**  
Generates a diff of changes between the working copy and the index or HEAD commit for a specific file or all files if no file is specified.

**Parameters:**  
- `repo_path` (str): The root path of the repository.  
- `file_path` (str, optional): Specific file path to diff; if omitted, diffs all changed files.

**Operation:**  
1. Retrieves the blob content for the file from the index or HEAD.  
2. Reads the current file content from the working directory.  
3. Compares the two versions line-by-line to produce a unified diff.  
4. Returns the diff output as a string.

**Example Usage:**

```python
# Diff all changes
print(diff_working_copy('/path/to/repo'))

# Diff specific file
print(diff_working_copy('/path/to/repo', 'src/main.py'))
```

---

## ASCII Diagram: Index and Working Copy Relationship

```
+----------------------+       +---------------------+       +----------------------+
|   Working Directory   |  <--  |      Git Index      |  <--  |     HEAD Commit      |
| (unstaged files)      |       | (staging area)      |       | (last commit state)  |
+----------------------+       +---------------------+       +----------------------+
          |                              |                              |
          |  Stage files (add/update)   |                              |
          +---------------------------->+                              |
          |                              |                              |
          |   Diff working copy          |                              |
          +---------------------------->+                              |
          |                              |                              |
          |                              |      Commit creates new      |
          |                              |      HEAD commit from index  |
          |                              |<-----------------------------+
          |                              |                              |
```

This diagram illustrates the flow of file changes from the working directory through the index to the HEAD commit, highlighting the staging and commit process.

---

**See Also:**  
- [`commit.md`](../Commit%20Management/commit.md) - For details on creating commits from the staged index.  
- [`status_and_diff.md`](../Status%20and%20Diff%20Utilities/status_and_diff.md) - For additional status and diff functionality.  
- [`objects.md`](../Object%20Management/objects.md) - For underlying object storage mechanisms used when staging files.