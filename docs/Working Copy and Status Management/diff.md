```mdx
# Diff Functionality Documentation

## Overview

This document provides a detailed explanation of the **diff** functionality within the git working copy and index management context. The diff feature is essential for identifying changes between the files staged in the git index and the files present in the working directory. It shows what modifications have been made to tracked files since they were last staged, enabling users and tools to inspect uncommitted changes before committing them. This file fits within the broader *Working Copy and Status Management* section of the documentation, complementing status reporting and file staging processes.

---

## Understanding Diff in Git

The diff operation compares the content of a file as recorded in the git index with the current content of the file in the working directory. This comparison highlights additions, deletions, and modifications at the line level, helping developers review changes.

### Key Concepts

- **Index (Staging Area):** The snapshot of files as they will appear in the next commit.
- **Working Copy:** The actual files on the filesystem that may be edited but not yet staged.
- **Diff Output:** A line-by-line summary of changes between the index and the working copy.

### ASCII Diagram: Relationship between Index and Working Copy

```
Working Copy (current files)
       |
       |   diff compares
       v
Index (staged snapshot)  <---->  Repository (committed history)
```

---

## Important Functions

### `diff(path: str) -> str`

#### Purpose

The `diff` function generates a textual representation of differences between the file in the index and the corresponding file in the working directory at the specified path.

#### Parameters

- `path` (str): The relative path to the file whose differences should be computed.

#### Preconditions

- The file must be tracked in the git index.
- Both index and working copy files must exist; otherwise, the function may raise an error or indicate the file is new/deleted.

#### How It Works

1. **Read Index Entry:** Retrieve the blob hash and metadata for the file from the index.
2. **Load Indexed Content:** Using the blob hash, read the file contents as recorded in the index.
3. **Load Working Copy Content:** Read the current file content from disk.
4. **Compute Differences:** Perform a line-by-line comparison between the index content and the working copy content.
5. **Format Output:** Construct a diff output string showing added (+), removed (-), and unchanged lines in a unified diff format.

#### Example Usage

```python
from pygit import diff

file_path = "src/main.py"
diff_output = diff(file_path)
print(diff_output)
```

*Sample output snippet:*

```diff
--- a/src/main.py
+++ b/src/main.py
@@ -10,7 +10,8 @@
-def old_function():
-    print("Old implementation")
+def new_function():
+    print("New implementation with fixes")
```

---

### Supporting Function: `get_index_content(path: str) -> str`

#### Purpose

Fetches the content of a file as currently staged in the git index.

#### Parameters

- `path` (str): Path to the indexed file.

#### Operation

- Looks up the blob hash for the file in the index.
- Reads and returns the content associated with that blob.

#### Example Usage

```python
content = get_index_content("README.md")
print(content)
```

---

### Supporting Function: `get_working_copy_content(path: str) -> str`

#### Purpose

Reads the current content of a file from the working directory.

#### Parameters

- `path` (str): Path to the working copy file.

#### Operation

- Opens and reads the file from disk.
- Returns the raw content as a string.

#### Example Usage

```python
content = get_working_copy_content("README.md")
print(content)
```

---

## Summary

The **diff** functionality is a critical part of the workflow for inspecting changes before committing. By comparing the index and working copy, it provides a clear, line-based summary of what has changed, enabling developers to make informed decisions about staging and committing files.

For related information on file status detection, see the [status.md](./status.md) documentation file in this section.

```