# git_operations.md

## Overview

This document provides comprehensive reference material for the core Git commands implemented in the codebase, including `add`, `commit`, `status`, and `diff`. These commands are fundamental to managing the repository state, enabling users to stage changes, record snapshots of the repository, inspect current status, and view differences between various states. This file fits within the **Basic Git Operations** section of the documentation tree, complementing more focused documents on commit internals, status scanning, and diff generation.

The commands and helper functions documented here serve as the interface between the user’s working copy and the underlying repository structures, orchestrating interactions with the index, object storage, and branch references.

---

## `add` Command

### Purpose

The `add` command stages changes from the working directory into the Git index (staging area). It updates the index with the current content of specified files, preparing them to be included in the next commit.

### Parameters

- **file_paths** (`List[str]`): Paths of files to stage relative to the repository root.

### Operation

1. Reads each specified file from the working directory.
2. Hashes the file content to create a blob object.
3. Stores the blob in the object database if not already present.
4. Updates the index entries to point to the new blob hashes.
5. Writes the updated index back to disk.

### Example Usage

```python
# Stage two files before committing
add(['README.md', 'src/main.py'])
```

---

## `commit` Command

### Purpose

The `commit` command records a snapshot of the current state of the index into the repository history. This creates a new commit object pointing to a tree representing the staged files.

### Parameters

- **message** (`str`): Commit message describing the changes.
- **author** (`str`): Author information, e.g., "Jane Doe <jane@example.com>".
- **committer** (`str`): Committer information, often same as author.
- **parent_commit_hash** (`str`, optional): Hash of the parent commit, if any.

### Operation

1. Builds a tree object from the current index entries, representing the staged files.
2. Creates a commit object with metadata (author, committer, message, parent commit).
3. Writes the commit object to the object database.
4. Updates the branch reference (e.g., `refs/heads/master`) to point to the new commit hash.

### Example Usage

```python
commit(
    message="Add initial project files",
    author="Jane Doe <jane@example.com>",
    committer="Jane Doe <jane@example.com>",
    parent_commit_hash="a1b2c3d4"
)
```

---

## `status` Command

### Purpose

The `status` command reports the state of the working directory and index relative to the last commit. It identifies changed, added, deleted, and untracked files.

### Parameters

- None explicitly; operates on the current repository state.

### Operation

1. Reads the HEAD commit and its associated tree.
2. Reads the current index entries.
3. Scans the working directory files and computes their hashes.
4. Compares working directory, index, and HEAD to classify file states:
   - **Modified**: File changed but staged differently.
   - **Staged**: File changes staged for the next commit.
   - **Untracked**: Files not present in index or HEAD.
5. Outputs a categorized list of files with their current status.

### Example Usage

```python
status()
# Output might be:
#   modified: src/main.py
#   new file: tests/test_main.py
#   deleted: docs/old_doc.md
```

---

## `diff` Command

### Purpose

The `diff` command generates a unified diff output showing differences between two states, commonly between the working directory and the index or between the index and the last commit.

### Parameters

- **from_tree** (`dict`): Tree object or snapshot representing the "base" state.
- **to_tree** (`dict`): Tree object or snapshot representing the "target" state.
- **paths** (`List[str]`, optional): Limit diff output to specific files.

### Operation

1. For each file in the union of `from_tree` and `to_tree`:
   - Reads file content from both states.
   - Computes line-by-line differences.
2. Formats the differences in unified diff format.
3. Outputs the diff as text.

### Example Usage

```python
diff(from_tree=commit_tree_hash, to_tree=index_tree_hash, paths=['src/main.py'])
```

### Example Unified Diff Output

```
--- a/src/main.py
+++ b/src/main.py
@@ -10,7 +10,8 @@
-def old_function():
-    pass
+def new_function():
+    print("Updated!")
```

---

## ASCII Diagram: Git Command Interactions Overview

```
+-------------------+
|   Working Copy    |
+---------+---------+
          |
          | git add
          v
+---------+---------+
|      Index (Staging)     |
+---------+---------+
          |
          | git commit
          v
+---------+---------+
|      Object Store       |
|  (Blobs, Trees, Commits)|
+---------+---------+
          |
          | git push (not covered here)
          v
+-------------------+
|   Remote Repository  |
+-------------------+

git status and git diff commands inspect:
- Working copy vs Index
- Index vs Last Commit (HEAD)
```

---

This completes the reference documentation for the core Git operations implemented in this module. For deeper details on commit internals, index management, and diff algorithms, see the related files:

- [commit.md](commit.md)
- [index_management.md](index_management.md)
- [status.md](status.md)
- [diff.md](diff.md)