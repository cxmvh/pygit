# status.md

## Overview

This document provides an in-depth overview of the `status` command within the Git implementation, focusing on how it determines the state of the working copy in relation to the repository’s index and object database. The `status` command is essential for developers to quickly understand which files have been modified, added, or deleted, and whether those changes are staged for commit. This documentation fits into the broader **Working Copy Status and Diff** section of the project, complementing related commands such as `diff` and `ls_files`. It elaborates on the internal processes of working copy scanning, index reading, object hashing, and change detection mechanisms.

---

## Functions

### `status(repo_path: str) -> StatusReport`

#### Purpose

The `status` function is the main entry point for generating a summary of the working copy's state relative to the Git index and repository objects. It identifies changes in tracked files, untracked files, and staged changes, enabling users to understand what actions are needed before committing.

#### Parameters

- `repo_path` (str): The filesystem path to the root of the Git repository. This path is used to locate the working copy, `.git` directory, and index file.

#### Operation

1. **Index Reading:**  
   The function reads the Git index file located inside the `.git` directory. This index contains metadata about files currently staged for commit, including file paths, timestamps, and hashed content identifiers.

2. **Working Copy Scanning:**  
   It scans the working directory recursively to enumerate files and their current state, including modification times and content hashes.

3. **Hashing Files:**  
   For files in the working directory, their contents are hashed (typically using SHA-1) to generate unique object identifiers. This step is critical for detecting content changes beyond simple timestamp differences.

4. **Change Detection:**  
   By comparing the index entries and working copy hashes, the function determines:
   - Which files have been modified but not staged.
   - Which files are staged and differ from the last commit.
   - Which files are untracked (present in working copy but absent from the index).
   - Which files have been deleted.

5. **Status Reporting:**  
   The function compiles a `StatusReport` object encapsulating these differences, which can then be presented to the user or used by other commands like `commit` or `diff`.

#### Example Usage

```python
from pygit import status

repo_path = "/path/to/my/repo"
report = status(repo_path)

print("Modified files (not staged):")
for file_path in report.modified:
    print(f"  {file_path}")

print("Staged files:")
for file_path in report.staged:
    print(f"  {file_path}")

print("Untracked files:")
for file_path in report.untracked:
    print(f"  {file_path}")
```

---

### Supporting Functions

#### `_read_index(index_path: str) -> IndexEntries`

Reads and parses the Git index file to retrieve the current staging information.

- **Parameters:**  
  - `index_path` (str): Full path to the index file (usually `.git/index`).

- **Operation:**  
  Parses the binary index file format to extract file entries, including their paths, stat info, and SHA-1 hashes of their contents.

- **Returns:**  
  An `IndexEntries` object representing all staged files.

---

#### `_hash_file(file_path: str) -> str`

Computes the SHA-1 hash of the file’s contents.

- **Parameters:**  
  - `file_path` (str): Path to the file to be hashed.

- **Operation:**  
  Reads the file in binary mode and computes the SHA-1 hash consistent with Git’s object hashing rules.

- **Returns:**  
  A hexadecimal SHA-1 hash string representing the file content.

---

#### `_scan_working_copy(root_path: str) -> Dict[str, FileInfo]`

Recursively scans the working directory for files and gathers their metadata.

- **Parameters:**  
  - `root_path` (str): Path to the repository’s working directory root.

- **Operation:**  
  Walks through the directory tree, ignoring files and directories as per `.gitignore` rules if implemented, and collects file paths along with modification times and file sizes.

- **Returns:**  
  A dictionary mapping file paths (relative to `root_path`) to `FileInfo` objects.

---

### ASCII Diagram: Status Command Workflow

```
+---------------------+
|  Working Directory   |
| (Files and folders)  |
+----------+----------+
           |
           | Scan files and metadata
           v
+---------------------+
|    Working Copy     |
|    File Info Map    |
+----------+----------+
           |
+---------------------+
|  Read Git Index     |
|  (.git/index file)  |
+----------+----------+
           |
           | Compare file hashes and timestamps
           v
+---------------------+
|   Change Detection  |
| - Modified          |
| - Staged            |
| - Untracked         |
| - Deleted           |
+----------+----------+
           |
           v
+---------------------+
|   Status Report     |
| (Summary for user)  |
+---------------------+
```

---

## Notes

- The `status` command depends heavily on accurate hashing and index reading; performance optimizations often involve caching file metadata and hashes.
- Integration with `.gitignore` patterns and handling of submodules may extend this functionality but are outside the scope of this core documentation.
- For a comprehensive understanding of related operations, refer to:
  - [`index_management.md`](../Index%20Management/index_management.md) for detailed index file handling.
  - [`diff.md`](diff.md) for how differences between file versions are computed and displayed.

---

This document is part of the **Working Copy Status and Diff** section, providing fundamental insights into repository state detection and user feedback mechanisms essential for day-to-day Git operations.