# Git Index Handling Documentation

## Overview

This document provides a detailed reference for the git index handling functionalities, integral to managing the staging area in the git repository. The index is a critical data structure that tracks the files slated for the next commit, storing metadata and file content hashes. This file (`index.md`) focuses on reading and writing index entries, managing file tracking in the working copy, and related operations that help maintain the coherence between the working directory and the git object database. It fits within the "Working Copy Status and Diff" section of the broader documentation tree, closely tied to status reporting and commit creation workflows.

---

## Function Documentation

### `read_index()`

**Purpose:**  
Reads the git index file (`.git/index`) and returns a list of `IndexEntry` objects representing the current staging area entries.

**Parameters:**  
None

**Returns:**  
- `List[IndexEntry]`: A list of index entries, each containing file metadata and SHA-1 hash of the staged content.

**Operation:**  
- Attempts to read binary data from `.git/index`.
- Verifies the file signature (`DIRC`) and version (2).
- Validates integrity with a checksum.
- Iteratively parses each index entry (including file stats, SHA-1, and file path).
- Handles alignment and padding for each entry.
- Returns all parsed entries.

**Example usage:**  
```python
entries = read_index()
for entry in entries:
    print(f"Path: {entry.path}, SHA-1: {entry.sha1.hex()}")
```

---

### `write_index(entries)`

**Purpose:**  
Writes a list of `IndexEntry` objects back to the git index file, updating the staging area.

**Parameters:**  
- `entries` (List[IndexEntry]): The index entries to write to the index file.

**Operation:**  
- Packs each entry's metadata and path into binary format.
- Adds padding for alignment.
- Writes header including signature, version, and entry count.
- Computes and appends SHA-1 checksum of the index content.
- Writes the complete binary data to `.git/index`.

**Example usage:**  
```python
entries = read_index()
# Modify entries as needed
write_index(entries)
```

---

### `add(paths)`

**Purpose:**  
Adds one or more files to the git index, staging them for commit.

**Parameters:**  
- `paths` (List[str]): List of file paths to add to the index.

**Operation:**  
- Normalizes file paths to use forward slashes.
- Reads the current index entries.
- Removes any entries that correspond to the given paths to avoid duplicates.
- For each path:
  - Reads file contents.
  - Hashes the content as a blob object, writing it to the object store.
  - Retrieves file metadata (timestamps, mode, UID, GID, size).
  - Creates a new `IndexEntry` with this information and the blob SHA-1.
- Sorts all entries by path.
- Writes updated entries back to the index.

**Example usage:**  
```python
add(['README.md', 'src/main.py'])
```

---

### `ls_files(details=False)`

**Purpose:**  
Lists files currently staged in the index.

**Parameters:**  
- `details` (bool, optional): If `True`, prints detailed information including file mode, SHA-1, and stage number.

**Operation:**  
- Reads index entries.
- For each entry, prints the file path.
- If `details` is enabled, also prints mode (octal), SHA-1, and stage.

**Example usage:**  
```python
ls_files()                # Lists file paths
ls_files(details=True)    # Lists detailed info
```

---

### `status()`

**Purpose:**  
Displays the status of the working copy by showing which files have changed, are new, or deleted compared to the index.

**Parameters:**  
None

**Operation:**  
- Calls `get_status()` to categorize files into changed, new, and deleted.
- Prints the categorized files in a user-readable format.

**Example usage:**  
```python
status()
# Output:
# changed files:
#    src/main.py
# new files:
#    docs/usage.md
# deleted files:
#    old_script.py
```

---

### `get_status()`

**Purpose:**  
Determines the status of the working copy by comparing the current files on disk against the index.

**Parameters:**  
None

**Returns:**  
- Tuple of three lists `(changed_paths, new_paths, deleted_paths)`.

**Operation:**  
- Walks the working directory, excluding `.git`, collecting all file paths.
- Reads the index entries and maps them by path.
- Compares file contents using SHA-1 hashes of blobs.
- Determines which files are changed, new, or deleted.

**Example usage:**  
```python
changed, new, deleted = get_status()
print("Changed:", changed)
print("New:", new)
print("Deleted:", deleted)
```

---

### `read_file(path)`

**Purpose:**  
Reads the contents of a file as bytes.

**Parameters:**  
- `path` (str): Path to the file.

**Returns:**  
- `bytes`: Contents of the file.

**Operation:**  
- Opens the file in binary mode and reads all data.

**Example usage:**  
```python
data = read_file('README.md')
print(data.decode())  # Print file contents as string
```

---

### `hash_object(data, obj_type, write=True)`

**Purpose:**  
Creates a git object by hashing the data and optionally writing it to the object store.

**Parameters:**  
- `data` (bytes): Content to hash.
- `obj_type` (str): Type of the object (e.g., `'blob'`, `'tree'`, `'commit'`).
- `write` (bool, optional): Whether to write the object to the object store.

**Returns:**  
- `str`: SHA-1 hash of the object as hex string.

**Operation:**  
- Prepares the object header with type and size.
- Concatenates header and data, then hashes with SHA-1.
- If `write` is `True`, compresses and writes the object to `.git/objects`.
  
**Example usage:**  
```python
sha1 = hash_object(b'hello world\n', 'blob')
print(f"Object SHA-1: {sha1}")
```

---

### `write_tree()`

**Purpose:**  
Creates a tree object from the current index entries, representing the state of the directory.

**Parameters:**  
None

**Returns:**  
- `str`: SHA-1 hash of the created tree object.

**Operation:**  
- Reads the index entries.
- For each entry, constructs a tree entry with mode and path.
- Concatenates all tree entries and hashes to create a tree object.

**Example usage:**  
```python
tree_sha1 = write_tree()
print(f"Tree object SHA-1: {tree_sha1}")
```

---

### `commit(message, author=None)`

**Purpose:**  
Commits the current state of the index to the local master branch with a commit message.

**Parameters:**  
- `message` (str): Commit message.
- `author` (str, optional): Author string in the format "Name <email>". If omitted, uses environment variables.

**Returns:**  
- `str`: SHA-1 hash of the commit object.

**Operation:**  
- Calls `write_tree()` to create a tree object from the index.
- Retrieves the current master commit hash as the parent (if any).
- Builds commit metadata including author, committer, timestamps.
- Creates commit object data and hashes it.
- Writes commit hash to `.git/refs/heads/master`.
- Prints confirmation.

**Example usage:**  
```python
commit_hash = commit("Initial commit")
print(f"Committed: {commit_hash}")
```

---

### ASCII Diagram: Overview of Git Index Flow

```
Working Directory
      |
      | (read files)
      v
+-----------------+       +-----------------+
|   read_file()   |       |   read_index()  |
+-----------------+       +-----------------+
      |                         |
      |                         |
      |                         v
      |               List of IndexEntry objects
      |                         |
      |                         |
      +-----------+-------------+
                  |
               add() / update
                  |
                  v
           +--------------+
           | write_index()| --> writes updated index file
           +--------------+
                  |
                  v
            Commit Flow
              (write_tree())
                  |
                  v
             commit()
```

---

# Summary

The functions documented here serve as the foundation for git's staging area management. They allow reading and writing the index file, adding files to the index, listing staged files, detecting changes relative to the working directory, and committing staged changes. Understanding these functions is essential for grasping how git tracks and records changes before they are committed to the repository history.