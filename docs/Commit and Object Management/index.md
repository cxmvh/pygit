# Index File Operations Documentation

## Overview

The `index.md` file documents the operations related to the Git index file, a crucial component in Git's version control system. The index acts as a staging area that tracks the snapshot of the working directory's files, bridging the gap between the working directory and the repository's commit history. This file focuses on reading from and writing to the index, managing index entries, and integrating these operations with core Git functionalities such as adding files, committing changes, and displaying status or diffs.

Situated within the broader "Commit and Object Management" section of the documentation tree, this file supports commands and processes that rely on the index to accurately reflect the current state of the working directory. Its functions are heavily interlinked with object handling, commit creation, and status reporting, making it a fundamental piece in understanding and manipulating Git's internal state.

---

## Functions

### 1. `read_index()`

**Purpose:**  
Reads the Git index file (`.git/index`) and returns a list of `IndexEntry` objects representing each entry in the index.

**Parameters:**  
None

**Returns:**  
`List[IndexEntry]` - List of index entries parsed from the index file.

**Operation:**  
- Attempts to read the raw index file bytes.
- Verifies the SHA-1 checksum at the end of the index file to ensure integrity.
- Unpacks the header to confirm the signature, version, and number of entries.
- Iterates through the binary data to unpack each index entry, decoding metadata and file path.
- Returns the assembled list of entries.

**Example:**
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### 2. `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the Git index file, updating the staging area.

**Parameters:**  
- `entries` (`List[IndexEntry]`): The list of index entries to write.

**Operation:**  
- Packs each `IndexEntry` into the Git index binary format.
- Aligns each entry to an 8-byte boundary with null padding.
- Constructs a header with the signature `"DIRC"`, version number, and entry count.
- Concatenates all entries and appends a SHA-1 checksum of the entire content.
- Writes the final data to `.git/index`.

**Example:**
```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### 3. `add(paths)`

**Purpose:**  
Adds files specified by `paths` to the Git index (staging area).

**Parameters:**  
- `paths` (`List[str]`): List of file paths to add to the index.

**Operation:**  
- Normalizes file paths to use forward slashes.
- Reads the current index entries and filters out any existing entries for the supplied paths.
- For each path:
  - Reads the file content and hashes it as a blob object.
  - Gathers file metadata (timestamps, device, inode, mode, owner IDs, size).
  - Creates a new `IndexEntry` with the metadata and SHA-1.
- Sorts all entries by path and writes the updated list back to the index.

**Example:**
```python
add(['README.md', 'src/main.py'])
```

---

### 4. `write_tree()`

**Purpose:**  
Writes a tree object based on the current index entries, representing a snapshot of the directory tree.

**Parameters:**  
None

**Returns:**  
`str` - SHA-1 hash of the created tree object.

**Operation:**  
- Reads the current index entries.
- For each entry, asserts that paths are top-level (no nested directories supported).
- Formats each entry as a tree entry with mode, path, and SHA-1.
- Concatenates all entries and hashes them as a tree object.
- Returns the tree object's SHA-1.

**Example:**
```python
tree_sha1 = write_tree()
print(f"Tree SHA-1: {tree_sha1}")
```

---

### 5. `commit(message, author=None)`

**Purpose:**  
Creates a commit object from the current index state and writes it to the repository, updating the `master` branch.

**Parameters:**  
- `message` (`str`): Commit message.
- `author` (`str`, optional): Author string in format `"Name <email>"`. Defaults to environment variables.

**Returns:**  
`str` - SHA-1 hash of the created commit object.

**Operation:**  
- Calls `write_tree()` to create a tree object representing the current index.
- Retrieves the current commit hash of `master` as the parent.
- Builds commit metadata including author and committer with timestamps.
- Formats commit content with tree, parent (if any), author, committer, and message.
- Hashes the commit object and writes it.
- Updates the `refs/heads/master` file with the new commit SHA-1.

**Example:**
```python
commit_sha1 = commit("Initial commit")
print(f"Committed with SHA-1: {commit_sha1}")
```

---

### 6. `get_status()`

**Purpose:**  
Returns the status of the working copy by comparing the working directory files and the index.

**Parameters:**  
None

**Returns:**  
`Tuple[List[str], List[str], List[str]]` - Tuple containing lists of changed, new, and deleted file paths.

**Operation:**  
- Walks the working directory tree, excluding `.git`.
- Collects all file paths present in the working directory.
- Reads the index entries and creates a map by path.
- Calculates:
  - Changed files: present in both working directory and index but with different contents.
  - New files: present in working directory but not in index.
  - Deleted files: present in index but missing in working directory.

**Example:**
```python
changed, new, deleted = get_status()
print("Changed files:", changed)
print("New files:", new)
print("Deleted files:", deleted)
```

---

### 7. `status()`

**Purpose:**  
Prints a human-readable status report of the working copy.

**Parameters:**  
None

**Operation:**  
- Calls `get_status()` to retrieve file status.
- Prints lists of changed, new, and deleted files neatly.

**Example:**
```python
status()
# Output:
# changed files:
#    README.md
# new files:
#    src/utils.py
# deleted files:
#    old_script.py
```

---

### 8. `diff()`

**Purpose:**  
Displays the differences between files in the index and the working directory.

**Parameters:**  
None

**Operation:**  
- Uses `get_status()` to find changed files.
- Reads the blob objects of these files from the index.
- Reads current working directory file contents.
- Uses `difflib.unified_diff` to generate and print diffs line-by-line.

**Example:**
```python
diff()
# Outputs unified diff for all changed files
```

---

### 9. `ls_files(details=False)`

**Purpose:**  
Lists all files currently in the Git index.

**Parameters:**  
- `details` (`bool`): If `True`, includes mode, SHA-1, and stage number alongside file paths.

**Operation:**  
- Reads index entries.
- Prints file paths, optionally with detailed info.

**Example:**
```python
ls_files(details=True)
# Output:
# 100644 e69de29bb2d1d6434b8b29ae775ad8c2e48c5391 0    README.md
# 100755 a3c6d4d1b2c9a8d76f0a4de7c5c8e258f1a7e9da 0    script.sh
```

---

## ASCII Diagram: Git Index File Structure

```
+------------------+
| Header           |
| - Signature "DIRC"|
| - Version (2)     |
| - Entry count     |
+------------------+
| Entry 1          |
| - Metadata (62 bytes) |
| - Path (variable) |
| - Padding (to 8-byte boundary) |
+------------------+
| Entry 2          |
| - Metadata       |
| - Path           |
| - Padding        |
+------------------+
| ...              |
+------------------+
| SHA-1 checksum (20 bytes) |
+------------------+
```

---

# Summary

This document covers the essential functions for managing Git's index file, enabling reading, writing, and manipulating index entries. These operations facilitate staging changes, creating tree objects, committing snapshots, and inspecting the working directory state. Together, they provide the foundation for higher-level Git commands that manage project history and collaboration.