# status.md - Working Copy Status and Diff

## Overview

This module provides essential functionality for reporting the status of the working copy in a Git repository and for displaying differences between the working copy and the Git index. It includes functions to detect changed, new, and deleted files, list the current status in a human-readable format, and show unified diffs for changed files.

This file is part of the **Working Copy Status and Diff** section within the pygit documentation tree, which focuses on inspecting and managing the state of the working directory relative to the index. It complements other files that handle index management, tree and commit objects, and diff generation.

Key functions documented here are:

- `get_status()` – Determine which files have changed, are new, or deleted.
- `status()` – Display the working copy status to the user.
- `diff()` – Show line-by-line differences for changed files between the index and the working copy.

These functions rely on other core pygit components such as reading the index, hashing objects, and reading object contents, enabling a comprehensive status and diff reporting toolset.

---

## Functions

### `get_status()`

Retrieve the current status of the working copy by comparing the files in the working directory against the Git index.

**Purpose:**

- Identify files that have been modified (`changed`).
- Identify files that are new and not yet added to the index (`new`).
- Identify files that have been deleted from the working directory but still exist in the index (`deleted`).

**Parameters:** None

**Returns:**

- A tuple of three lists `(changed_paths, new_paths, deleted_paths)` each sorted alphabetically.

**How it works:**

1. Walk through the working directory recursively, ignoring `.git` directories, and collect all file paths.
2. Read the current Git index entries and map them by file path.
3. Compute SHA-1 hashes of working copy files and compare them against the stored SHA-1 in the index to find files with changed content.
4. Determine which files exist only in the working directory (`new`) or only in the index (`deleted`).

**Example:**

```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### `status()`

Prints a human-readable summary of the working copy status to the console.

**Purpose:**

- Display lists of changed, new, and deleted files for the user.

**Parameters:** None

**Behavior:**

- Calls `get_status()` to retrieve the status.
- Prints each category only if it contains files.
- Indents file paths for readability.

**Example output:**

```
changed files:
    src/main.py
    README.md
new files:
    docs/usage.md
deleted files:
    old_script.py
```

**Example usage:**

```python
status()
```

---

### `diff()`

Show the unified diff of all changed files between the index and the working copy.

**Purpose:**

- Provide a line-by-line comparison of changes made to files since they were last added to the index.
- Helps users review modifications before committing.

**Parameters:** None

**How it works:**

1. Uses `get_status()` to identify changed files.
2. Reads the version of each changed file as stored in the index (the blob object).
3. Reads the current working copy file content.
4. Uses Python's `difflib.unified_diff` to generate a diff showing changes from the indexed version to the working copy.
5. Prints the diff with clear headers indicating file names and sources.

**Example usage:**

```python
diff()
```

**Sample output snippet:**

```
--- src/main.py (index)
+++ src/main.py (working copy)
@@ -10,7 +10,9 @@
-def foo():
-    print("Hello world")
+def foo():
+    print("Hello, world!")
+    print("Added a new line")
```

---

## ASCII Diagram: Status and Diff Flow

```
+----------------------------+
|   Working Directory Files   |
+-------------+--------------+
              |
              v
+----------------------------+       +-----------------+
|       get_status()          |<----->|   Git Index     |
+----------------------------+       +-----------------+
          |        |        |
          |        |        |
          v        v        v
    Changed    New Files   Deleted
          |        |        |
          +--------+--------+
                   |
                   v
           +----------------+
           |    status()    |  -- prints status summary
           +----------------+
                   |
                   v
           +----------------+
           |     diff()     |  -- prints diffs for changed files
           +----------------+
```

---

## Additional Notes

- `get_status()` depends on reading the Git index (`read_index()`), hashing file contents (`hash_object()`), and reading files (`read_file()`).
- The `diff()` function assumes the index stores blob objects representing file contents.
- The `status()` function is a user-friendly wrapper around `get_status()`.
- These functions exclude files in the `.git` directory from consideration.
- Changes are detected purely by comparing SHA-1 hashes of file contents, not by timestamp or other metadata.

---

# End of status.md Documentation