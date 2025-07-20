---
sidebar_position: 2
---

# Inspecting Working Copy Changes, Diffing, and Computing Status

## Overview

This document details the mechanisms used to inspect changes in the working copy of a git repository by comparing the current files with the index. It focuses on the diffing process between the index and the working copy, and the computation of file status to determine which files are modified, added, deleted, or untracked. This file fits within the "Repository Workflows" section of the broader documentation tree, providing essential insights into how git detects changes before committing or other operations. Understanding these processes is critical for developers working on tools that interact with git repositories at a low level or for extending git functionalities.

## Key Functions

### `diff_index_working_copy`

#### Purpose

This function compares the state of files recorded in the index against the actual files in the working directory. It identifies differences such as content changes, file mode modifications, and new or deleted files. The output is a detailed diff structure or list that highlights these discrepancies.

#### Parameters

- `index`: The current index state, representing a snapshot of files staged for commit.
- `working_copy_path`: The path to the working directory containing the actual files.
- `diff_options` (optional): Parameters to customize the diff behavior (e.g., ignore whitespace, rename detection).

#### Operation Steps

1. Load the list of entries from the index.
2. Iterate over each index entry and read the corresponding file from the working directory.
3. Compare the file contents and metadata (e.g., file mode, size, timestamps).
4. Record files that differ in content or metadata.
5. Detect files present in the working directory but absent from the index (untracked files).
6. Return a structured summary of all detected differences.

#### Example Usage

```python
index = load_index('.git/index')
diffs = diff_index_working_copy(index, '/path/to/working/copy')
for diff in diffs:
    print(f"File: {diff.path} Status: {diff.status}")
```

---

### `compute_status`

#### Purpose

This function computes the status of the working copy relative to the index and possibly the HEAD commit. It categorizes files into states such as modified, added, deleted, untracked, or unchanged. This status summary is fundamental for commands like `git status` that inform users about the current working state.

#### Parameters

- `index`: The repository’s current index.
- `working_copy_path`: Directory path of the working copy.
- `head_tree` (optional): The tree object from the HEAD commit for comparison.

#### Operation Steps

1. Invoke the diffing mechanism between the index and working copy.
2. Optionally, compare the index to the HEAD tree to detect staged changes.
3. Classify files into status categories based on differences found:
   - **Modified:** Files changed in the working copy but not staged.
   - **Staged:** Files changed and staged in the index.
   - **Untracked:** Files in the working directory not in the index.
   - **Deleted:** Files removed from the working directory or index.
4. Aggregate and return the status summary.

#### Example Usage

```python
index = load_index('.git/index')
head_tree = read_tree_from_commit('HEAD')
status = compute_status(index, '/path/to/working/copy', head_tree)
for file_status in status:
    print(f"{file_status.path}: {file_status.state}")
```

---

## ASCII Diagram: Status Computation Flow

```
+-------------------+
|   HEAD Commit     |
|   (HEAD Tree)     |
+---------+---------+
          |
          v
+-------------------+       +------------------+
|      Index        |<----->| Working Copy     |
| (Staged Snapshot) |       | (Actual Files)   |
+-------------------+       +------------------+
          |                          |
          |                          |
          +-----------+--------------+
                      |
                      v
               +--------------+
               | Status       |
               | Computation  |
               +--------------+
                      |
          +-----------+-------------+
          |                         |
  Modified / Staged          Untracked / Deleted
```

This diagram shows the relationship between the HEAD commit's tree, the index, and the working copy. The status computation compares these to produce a categorized summary of file states.

---

## Summary

This document provides reference material for inspecting the working copy's state by diffing it against the index and for computing the overall status of files in the repository. These operations are foundational for git's workflow commands and allow developers to programmatically determine what changes exist at any stage of the development cycle.