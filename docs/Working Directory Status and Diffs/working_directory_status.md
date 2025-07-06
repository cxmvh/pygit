# Working Directory Status Checks and Diff Generation

This document describes the mechanisms used to check the working directory status within the repository, how file changes are detected (new, modified, deleted), and how diffs are generated and displayed. It is part of the "Working Directory Status and Diffs" section of the documentation, which focuses on understanding the current state of files relative to the index and the last commit. This is crucial for enabling commands like `git status` and `git diff` that inform users about file changes before committing.

---

## Overview

The working directory status module is responsible for comparing the current files in the working directory with the repository’s index and HEAD commit tree. It detects:

- **New files**: Files present in the working directory but not in the index.
- **Modified files**: Files whose contents or metadata have changed since the last commit.
- **Deleted files**: Files previously tracked but removed from the working directory.

Additionally, it generates diffs showing line-by-line changes for modified files, enabling users to review changes before committing.

---

## Key Functions

### `get_working_directory_status(repo_path)`

#### Purpose

Determines the status of all files in the working directory relative to the index and HEAD commit. It returns a structured summary of new, modified, and deleted files.

#### Parameters

- `repo_path` (str): Path to the root of the repository.

#### Operation Steps

1. **Read HEAD commit tree**: Retrieve the snapshot of the repository at the last commit.
2. **Read the index**: Load the current index state, which tracks files staged for commit.
3. **Scan working directory**: Recursively list all files in the working directory.
4. **Compare files**:
   - Files in working directory but not in index → *New files*.
   - Files in both working directory and index but differing in content or metadata → *Modified files*.
   - Files in index but missing from working directory → *Deleted files*.
5. Return a dictionary or object summarizing these categories.

#### Example Usage

```python
status = get_working_directory_status('/path/to/repo')
print("New files:", status.new_files)
print("Modified files:", status.modified_files)
print("Deleted files:", status.deleted_files)
```

---

### `generate_diff(file_path, repo_path)`

#### Purpose

Generates a unified diff string showing line-by-line changes between the working directory version of a file and its version in the index or HEAD commit.

#### Parameters

- `file_path` (str): Relative path of the file to diff.
- `repo_path` (str): Path to the root of the repository.

#### Operation Steps

1. **Retrieve base version**:
   - If the file is tracked, load its content from the index or HEAD commit.
   - If new, base is empty.
2. **Read working directory version**: Load current content of the file.
3. **Compute diff**:
   - Use a line-based diff algorithm to find additions, deletions, and changes.
4. **Format diff output**:
   - Produce a unified diff string with hunk headers and line changes.
5. Return the diff string.

#### Example Usage

```python
diff_text = generate_diff('src/main.py', '/path/to/repo')
print(diff_text)
```

---

### `detect_file_changes(repo_path)`

#### Purpose

Helper function that returns detailed information for each file about its change status: new, modified, or deleted.

#### Parameters

- `repo_path` (str): Path to the root of the repository.

#### Operation Steps

1. Load index and HEAD tree.
2. Walk the working directory files.
3. For each file:
   - Check presence in index and HEAD.
   - Compare content hashes or timestamps.
4. Categorize each file accordingly.
5. Return a structured data set listing files by status.

---

## ASCII Diagram: File Status Workflow

```
+------------------+        +----------------+       +-------------------------+
| Working Directory |        |     Index      |       |       HEAD Commit       |
| (current files)   |        | (staged files) |       | (last commit snapshot)  |
+------------------+        +----------------+       +-------------------------+
          |                         |                           |
          |                         |                           |
          |                         |                           |
          +------------+------------+---------------------------+
                       |
              Compare files across these three states
                       |
      +----------------+----------------+-----------------+
      |                |                |                 |
  New files       Modified files   Deleted files     Unchanged files
(files in WD      (files present   (files missing    (files same in
 but not in       in WD & index    in WD but present index & WD)
 index)           but content or   in index)
                  metadata differs)
```

---

## Summary

This document explains how the system determines and reports changes in the working directory relative to the repository state. It covers detection of new, modified, and deleted files and details the generation of diffs that highlight content changes. These functions underpin the user-facing commands that present the status of the repository and enable staged commits.

For further details on related topics such as index management, see [index_management.md](../Index%20Management/index_management.md). For object handling and diff internals, refer to [object_handling.md](../Git%20Object%20and%20Storage%20Management/object_handling.md).