---
sidebar_position: 3
---

# Managing the Git Index and Working Copy

## Overview

This document covers the management of the Git index and working copy within a local repository. It explains how files are added to the index, how the index is read and written, how the status of the working copy is computed, how trees are written to represent directory states, and how files are listed. This file is part of the "Local Repository Internals" section, complementing documentation on repository initialization and object handling, and serves as a critical reference for understanding how Git tracks changes and prepares commits.

---

## Reading and Writing the Index

### Purpose

The Git index is a binary file that records the state of the working directory for the next commit. Reading and writing the index allows Git to keep track of which files are staged and their metadata.

### Description

- **Reading the Index:** Parses the `.git/index` file into an internal data structure representing tracked files, their modes, SHA-1 hashes, and other metadata.
- **Writing the Index:** Serializes the internal index data back into the `.git/index` file, ensuring consistency and atomic updates.

### Usage Example

```python
index = read_index(".git/index")
# Modify index entries as needed, e.g., add or remove files
write_index(".git/index", index)
```

---

## Adding Files to the Index

### Purpose

To stage changes, files from the working copy must be added to the index. This involves hashing file contents and updating index entries accordingly.

### Description

- Reads file content from the working directory.
- Computes the blob object's hash.
- Updates or adds the file entry in the index with the new SHA and metadata.

### Usage Example

```python
add_file_to_index("README.md", index, git_dir=".git")
write_index(".git/index", index)
```

---

## Computing the Status of the Working Copy

### Purpose

To determine which files have been modified, added, or deleted relative to the index and the last commit, enabling Git to report on changes.

### Description

- Compares the working directory files against the index entries.
- Detects untracked files, modified files, and deletions.
- Returns a summarized status indicating the state of each file.

### Usage Example

```python
status = compute_status(git_dir=".git", work_dir=".")
for file, state in status.items():
    print(f"{file}: {state}")
```

---

## Writing Trees

### Purpose

To record the state of the working directory or index in a Git tree object, representing directories and their contents for commits.

### Description

- Converts the current index or directory structure into a tree object.
- Recursively processes directories and their contents.
- Stores the tree object in the Git object database.

### ASCII Diagram

```
Tree Object Structure:

root_tree
├── file1.txt (blob)
├── src (tree)
│   ├── main.py (blob)
│   └── utils.py (blob)
└── README.md (blob)
```

### Usage Example

```python
tree_hash = write_tree(index, git_dir=".git")
print(f"Tree object created with hash: {tree_hash}")
```

---

## Listing Files Tracked in the Index

### Purpose

To retrieve a list of all files currently tracked in the index, useful for status reporting or other operations.

### Description

- Reads the index file.
- Extracts and returns file paths tracked by Git.

### Usage Example

```python
files = ls_files(git_dir=".git")
for f in files:
    print(f)
```

---

This documentation provides foundational knowledge for developers working with Git internals, specifically around how Git manages and tracks file changes before commits. For more details on repository initialization and object handling, see [repository_and_object_model.md](./repository_and_object_model.md) and [object_handling_and_display.md](./object_handling_and_display.md). For workflows related to commit creation and change inspection, refer to files in the "Repository Workflows" section.