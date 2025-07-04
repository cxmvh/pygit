# Status and Diff Utilities

## Overview

The `status_and_diff.md` file documents the functionality related to checking the status of the repository, listing changed files, and displaying diffs between various states of the repository. This file focuses primarily on the `status()` and `get_status()` functions along with their supporting utilities, which play a crucial role in providing users with clear insights into the current state of their working copy relative to the index and the last commit. This documentation is part of the broader **Status and Diff Utilities** section, which complements other areas such as index management and diffing mechanisms. Understanding these functions is essential for developers who want to implement or extend features related to change tracking and visualizing differences within the repository.

---

## Functions

### `status()`

#### Purpose

The `status()` function provides a comprehensive snapshot of the repository's current state by examining the working directory, the staging index, and the last committed tree. It identifies files that have been modified, added, deleted, or remain unchanged. This function is typically the entry point for users or higher-level commands that want to display or act upon the repository status.

#### Parameters

- `repo_path` (str): The path to the root directory of the repository.
- `show_untracked` (bool, optional): Whether to include untracked files in the status output. Defaults to `True`.
- `ignore_submodules` (bool, optional): Whether to ignore changes in submodules. Defaults to `True`.

#### Operation

1. **Read the Index:** Load the current staging index file to understand what files are staged for commit.
2. **Read the HEAD Commit Tree:** Determine the state of files in the last commit by reading the tree object pointed to by HEAD.
3. **Scan Working Directory:** Walk through the working directory to detect:
   - Modified files (content changes since last commit or staging)
   - Added files (new files not tracked yet)
   - Deleted files (files removed from working directory but still tracked)
4. **Compare States:** For each file, compare its state in the working directory, the index, and the HEAD commit tree to classify the file's status.
5. **Return Structured Status:** Provide a structured dictionary or list containing file paths grouped by their status categories such as `modified`, `added`, `deleted`, `untracked`, and `staged`.

#### Example Usage

```python
from pygit.status import status

repo_path = "/path/to/my/repo"
current_status = status(repo_path, show_untracked=True)

print("Modified files:")
for file_path in current_status['modified']:
    print(f"  {file_path}")

print("Untracked files:")
for file_path in current_status['untracked']:
    print(f"  {file_path}")
```

---

### `get_status()`

#### Purpose

The `get_status()` function serves as a lower-level utility that supports `status()` by performing detailed comparisons between the index and the working directory. It focuses specifically on the mechanics of detecting file changes and categorizing them into status codes.

#### Parameters

- `index` (Index): The parsed index data structure representing the staging area.
- `working_dir` (str): The path to the working directory.
- `head_tree` (Tree): The tree object representing the last committed state.
- `show_untracked` (bool): Flag to include untracked files.
- `ignore_submodules` (bool): Flag to skip submodule changes.

#### Operation

1. Iterate over entries in the index to check for differences with corresponding files in the working directory.
2. Detect modifications by comparing file metadata and contents.
3. Detect deletions where files exist in the index but are missing in the working directory.
4. Identify untracked files by scanning the working directory for files not present in the index.
5. Compile a detailed list or dictionary mapping file paths to their status codes (e.g., 'M' for modified, 'A' for added).

#### Example Usage

```python
from pygit.status import get_status
from pygit.index import read_index
from pygit.objects import read_tree

index = read_index("/path/to/my/repo/.git/index")
head_tree = read_tree("HEAD")

changes = get_status(index, "/path/to/my/repo", head_tree, show_untracked=True, ignore_submodules=True)

for filepath, status_code in changes.items():
    print(f"{filepath}: {status_code}")
```

---

### Supporting Functions Overview

While `status()` and `get_status()` form the core of the status detection functionality, several auxiliary functions assist by reading repository objects, comparing file contents, and formatting output for diff display. These include:

- **`read_index()`**: Reads and parses the Git index file.
- **`read_tree()`**: Retrieves the tree object for a given commit or reference.
- **`diff()`**: Computes the textual differences between two file versions.
- **`list_changed_files()`**: Lists files that have changed between two repository states.

---

## ASCII Diagram: Status Detection Flow

```
+-------------------------+
|       status()          |
+-----------+-------------+
            |
            v
+-------------------------+
|   Read HEAD Commit Tree  |
+-----------+-------------+
            |
            v
+-------------------------+
|       Read Index         |
+-----------+-------------+
            |
            v
+-------------------------+
|    Scan Working Dir      |
+-----------+-------------+
            |
            v
+-------------------------+
| Compare HEAD, Index, WD  |
+-----------+-------------+
            |
            v
+-------------------------+
|   Generate Status Report |
+-------------------------+
```

---

## Summary

The `status_and_diff.md` document equips developers with detailed insights into how the repository status is computed and how diffs are generated. By understanding the `status()` and `get_status()` functions, developers can better integrate change detection into their workflows, build custom status reporting tools, or enhance existing Git client features. This documentation file is a central reference for anyone working with repository state inspection and change visualization in the pygit project.