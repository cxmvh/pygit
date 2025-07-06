# diff.md

## Overview

This document explains the `diff` command implementation that compares the Git index (staging area) with the working copy files. It details how differences between the two are detected and presented as unified diffs, a standard format showing added, removed, and changed lines in a human-readable way. This file is part of the **Working Copy Status and Diff** section, complementing the `status.md` file by focusing specifically on the diff output generation. Understanding this command is crucial for developers who want to inspect changes before staging or committing, aiding in code review and version control.

---

## Functions

### `diff_index_to_working_copy(repo, index, path='')`

#### Purpose
This is the main function driving the diff operation. It compares the file contents recorded in the Git index against the current files in the working copy (the file system). It produces a unified diff output that highlights differences line-by-line.

#### Parameters
- `repo`: The repository object representing the current Git repository.
- `index`: The parsed Git index data structure containing file entries and their recorded blob hashes.
- `path` (optional): A prefix path used for relative file paths when recursively diffing directories or subtrees.

#### Operation Steps
1. **Iterate over each file entry in the index.**
2. **For each file, read the content from the working copy on disk.**
3. **Retrieve the blob content from the object database using the blob hash stored in the index.**
4. **Compare the two file contents line-by-line.**
5. **Generate a unified diff format output showing differences:**
   - Lines prefixed with `+` indicate additions in the working copy.
   - Lines prefixed with `-` indicate deletions compared to the index.
   - Context lines show unchanged lines for orientation.
6. **Accumulate diffs for all files and return or print the complete diff.**

#### Example Usage
```python
import pygit

repo = pygit.Repository('.git')
index = repo.read_index()

# Produce diff output comparing index and working copy
diff_output = pygit.diff_index_to_working_copy(repo, index)

print(diff_output)
```

This will print a unified diff showing all changes in the working copy relative to the staged snapshot.

---

### Supporting Concepts and Format

#### Unified Diff Format Overview

The unified diff format shows files with changes as follows:

```
--- a/<filename>
+++ b/<filename>
@@ -<start line>,<lines changed> +<start line>,<lines changed> @@
 <context lines>
 -<lines removed>
 +<lines added>
```

##### Example snippet

```
--- a/src/main.py
+++ b/src/main.py
@@ -10,7 +10,8 @@
 def example():
-    print("Hello world")
+    print("Hello, world!")
+    print("Additional line")
```

This means:
- The line `print("Hello world")` was replaced.
- One new line was added after it.

---

## ASCII Diagram: Diff Comparison Flow

```
+----------------+          +-----------------+          +--------------------+
|  Git Index     |          | Working Copy    |          | Diff Output        |
| (staged state) |          | (filesystem)    |          | (unified diff)     |
+----------------+          +-----------------+          +--------------------+
        |                           |                             ^
        | Blob hashes & paths       | File content                |
        |-------------------------->|                             |
        |                           |                             |
        | <-------------------------|                             |
        | Blob content from object db                              |
        |                           |                             |
        +-------------------------------------------------------->+
                       Compare line-by-line, generate diff
```

---

## Additional Notes

- This diff command is distinct from comparing commits or branches; it focuses on the staged snapshot versus the live working copy.
- It relies heavily on accurate index reading and blob retrieval from the Git object database.
- The output can be used by other commands such as `git status` or patch applications.

For more details on related topics, see:
- [status.md](./status.md) — for working copy scanning and change detection.
- [index_management.md](./index_management.md) — for reading and writing the Git index.

---

*End of diff.md*